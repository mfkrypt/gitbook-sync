# Windows Credential Manager

### <mark style="color:green;">What is Credentials Manager?</mark>

Credential Manager is a Windows feature that stores logon-sensitive information for websites, applications, and networks. It contains login credentials such as usernames, passwords, and internet addresses. There are four credential categories:

* Web credentials&#x20;
* Windows credentials such as NTLM or Kerberos.
* Clear-text usernames and passwords.
* Certificate-based credentials

***

### <mark style="color:green;">Accessing Credential Manager</mark>

We can use `vaultcmd` to list the current windows vaults available in the Windows target

```
vaultcmd /list
```

<figure><img src="../../../.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

By default, Windows has two vaults, one for Web and the other one for Windows machine credentials. The above output confirms that we have the two default vaults.

***

### <mark style="color:green;">Credential Dumping</mark>

The VaultCmd is not able to show the password, but we can rely on other PowerShell Scripts such as [Get-WebCredentials.ps1](https://github.com/samratashok/nishang/blob/master/Gather/Get-WebCredentials.ps1). Ensure to execute PowerShell with bypass policy to import it as a module as follows,

```
powershell -ex bypass
```

<figure><img src="../../../.gitbook/assets/image (677).png" alt=""><figcaption></figcaption></figure>

***

### <mark style="color:green;">RunAs</mark>

An alternative method of taking advantage of stored credentials is by using RunAs.  The `/savecred` argument allows you to save the credentials of the user in Windows Credentials Manager (under the Windows Credentials section). So, the next time we execute as the same user, runas will not ask for a password.

&#x20;Another way to enumerate stored credentials is by using `cmdkey`, which is a tool to create, delete, and display stored Windows credentials. By providing the `/list` argument, we can show all stored credentials, or we can specify the credential to display more details `/list:computername`

```powershell
cmdkey /list
```

<figure><img src="../../../.gitbook/assets/image (678).png" alt=""><figcaption></figcaption></figure>

We are going for the last one. The output shows that we have a domain password stored as the `thm\thm-local` user.

Now let's use runas to execute Windows applications as the `thm-local` user

```powershell
runas /savecred /user:THM.red\thm-local cmd.exe
```

A new cmd.exe pops up with a command prompt ready to use. Now run the whoami command to confirm that we are running under the desired user. There is a flag in the `c:\Users\thm-local\Saved Games\flag.txt`

<figure><img src="../../../.gitbook/assets/image (679).png" alt=""><figcaption></figcaption></figure>

***

### <mark style="color:green;">Mimikatz</mark>

Mimikatz can also be used to dump credentials manager using the `sekurlsa` module

```powershell
mimikatz.exe
mimikatz # privilege::debug
mimikatz # sekurlsa::credman
```

<figure><img src="../../../.gitbook/assets/image (680).png" alt=""><figcaption></figcaption></figure>

