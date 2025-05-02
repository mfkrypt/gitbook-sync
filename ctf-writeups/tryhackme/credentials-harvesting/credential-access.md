# Credential Access

Credential access is where adversaries may find credentials in compromised systems and gain access to user credentials. It helps adversaries to reuse them or impersonate the identity of a user. This is an important step for lateral movement and accessing other resources such as other applications or systems. Obtaining legitimate user credentials is preferred rather than exploiting systems using CVEs.

Credentials are stored insecurely in various locations in systems:

***

### <mark style="color:green;">Clear-Text Files</mark>

Attackers may search a compromised machine for credentials in local or remote file systems. Clear-text files could include sensitive information created by a user, containing passwords, private keys, etc.&#x20;

The following are some of the types of clear-text files that an attacker may be interested in:

* Commands history
* Configuration files (Web App, FTP files, etc.)
* Other Files related to Windows Applications (Internet Browsers, Email Clients, etc.)
* Backup files
* Shared files and folders
* Registry
* Source code&#x20;

Searching for "password" keyword in the Registry:

```powershell
C:\Users\user> reg query HKLM /f password /t REG_SZ /s
#OR
C:\Users\user> reg query HKCU /f password /t REG_SZ /s
```

<figure><img src="../../../.gitbook/assets/image (183).png" alt=""><figcaption></figcaption></figure>

***

### <mark style="color:green;">Password Managers</mark>

A password manager is an application to store and manage users' login information for local and Internet websites and services. Since it deals with users' data, it must be stored securely to prevent unauthorized access.&#x20;

Examples of Password Manager applications:

* Built-in password managers (Windows)
* Third-party: KeePass, 1Password, LastPass

However, misconfiguration and security flaws are found in these applications that let adversaries access stored data. Various tools could be used during the enumeration stage to get sensitive data in password manager applications used by Internet browsers and desktop applications.

***

### <mark style="color:green;">Memory Dump</mark>

The Operating system's memory is a rich source of sensitive information that belongs to the Windows OS, users, and other applications. Data gets loaded into memory at run time or during the execution. Thus, accessing memory is limited to administrator users who fully control the system.

The following are examples of memory stored sensitive data, including:

* Clear-text credentials
* Cached passwords
* AD Tickets

***

### <mark style="color:green;">Active Directory</mark>

Active Directory stores a lot of information related to users, groups, computers, etc. Thus, enumerating the Active Directory environment is one of the focuses of red team assessments. Active Directory has a solid design, but misconfiguration made by admins makes it vulnerable to various attacks shown in this room.

The following are some of the Active Directory misconfigurations that may leak users' credentials.

* Users' description: Administrators set a password in the description for new employees and leave it there, which makes the account vulnerable to unauthorized access.&#x20;
* Group Policy SYSVOL: Leaked encryption keys let attackers access administrator accounts. Check Task 8 for more information about the vulnerable version of SYSVOL.
* NTDS: Contains AD users' credentials, making it a target for attackers.
* AD Attacks: Misconfiguration makes AD vulnerable to various attacks, which we will discuss in Task 9.

<figure><img src="../../../.gitbook/assets/image (182).png" alt=""><figcaption></figcaption></figure>
