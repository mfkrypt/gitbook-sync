# Local Password Administrator Solution (LAPS)

### <mark style="color:green;">Group Policy Preferences (GPP)</mark>

GPP is a tool that allows administrators to create domain policies with embedded credentials. Once the GPP is deployed, different XML files are created in the **SYSVOL** folder. **SYSVOL** is an essential component of Active Directory and creates a shared directory on an NTFS volume that all authenticated domain users can access with reading permission.

The issue was the GPP relevant XML files contained a password encrypted using AES-256 bit encryption. At that time, the encryption was good enough until Microsoft somehow published its **private key** on [MSDN](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-gppref/2c15cbf0-f086-4c74-8b70-1f2fa45dd4be?redirectedfrom=MSDN). Since Domain users can read the content of the SYSVOL folder, it becomes easy to decrypt the stored passwords. One of the tools to crack the SYSVOL encrypted password is [Get-GPPPassword](https://github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Get-GPPPassword.ps1).

***

### <mark style="color:green;">Local Password Administrator Solution (LAPS)</mark>

In 2015, Microsoft removed storing the encrypted password in the SYSVOL folder. It introduced the Local Administrator Password Solution (LAPS), which offers a much more secure approach to remotely managing the local administrator password.

The new method includes two new attributes (ms-mcs-AdmPwd and ms-mcs-AdmPwdExpirationTime) of computer objects in the Active Directory. The <mark style="color:red;">**`ms-mcs-AdmPwd`**</mark> attribute contains a clear-text password of the local administrator, while the <mark style="color:red;">**`ms-mcs-AdmPwdExpirationTime`**</mark> contains the expiration time to reset the password. LAPS uses `admpwd.dll` to change the local administrator password and update the value of `ms-mcs-AdmPwd`.

<figure><img src="../../../.gitbook/assets/image (682).png" alt=""><figcaption></figcaption></figure>

***

### <mark style="color:green;">Enumerate for LAPS</mark>

let's start enumerating it. First, we check if LAPS is installed in the target machine, which can be done by checking the `admpwd.dll` path

<figure><img src="../../../.gitbook/assets/image (683).png" alt=""><figcaption></figcaption></figure>

The output confirms that we have LAPS on the machine. Let's check the available commands to use for `AdmPwd` cmdlets as follows.

```powershell
Get-Command *AdmPwd*
```

<figure><img src="../../../.gitbook/assets/image (684).png" alt=""><figcaption></figcaption></figure>

Now we enumerate available OUs

```powershell
Get-ADOrganizationalUnit -Filter *
```

<figure><img src="../../../.gitbook/assets/image (685).png" alt=""><figcaption></figcaption></figure>

We can see an available group here which is THMorg, safe to assume the OU has the "All extended rights" attribute that deals with LAPS. We will be using the "Find-AdmPwdExtendedRights" cmdlet to provide the right OU.

```powershell
Find-AdmPwdExtendedRights -Identity THMorg
```

<figure><img src="../../../.gitbook/assets/image (686).png" alt=""><figcaption></figcaption></figure>

The output shows that the `THMLAPsReader` group in `THMorg` has the right access to LAPS. Let's check the group and its members.

```powershell
net groups "LAPsReader"
```

<figure><img src="../../../.gitbook/assets/image (687).png" alt=""><figcaption></figcaption></figure>

***

### <mark style="color:green;">Getting the Password</mark>

We found that the `bk-admin` user is a member of `LAPsReader`, so in order to get the LAPS password, we need to compromise or impersonate the bk-admin user.

After compromising the right user, we can get the LAPS password using `Get-AdmPwdPassword` cmdlet by providing the target machine with LAPS enabled

