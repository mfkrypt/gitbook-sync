# Local Security Authority Subsystem Service (LSASS)

### <mark style="color:green;">What is the LSASS?</mark>

Local Security Authority Server Service (LSASS) is a Windows process that handles the operating system security policy and enforces it on a system. It verifies logged in accounts and ensures passwords, hashes, and Kerberos tickets. Windows system stores credentials in the LSASS process to enable users to access network resources, such as file shares, SharePoint sites, and other network services, without entering credentials every time a user connects.

***

### <mark style="color:green;">GUI</mark>

To dump any running Windows process using the GUI, open the Task Manager, and from the Details tab, find the required process, right-click on it, and select "Create dump file".

<figure><img src="../../../.gitbook/assets/image (167).png" alt=""><figcaption></figcaption></figure>

Once the dumping process is finished, a pop-up message will show containing the path of the dumped file. Now copy the file and transfer it to our Attacking Machine to crack the NTLM hashes

***

### <mark style="color:green;">Mimikatz</mark>

Mimikatz can also be utilized to dump the LSASS process. Remember that the LSASS process is running as a SYSTEM. Thus in order to access users' hashes, we need a system or local administrator permissions. Thus, open the command prompt and run it as administrator.

<figure><img src="../../../.gitbook/assets/image (173).png" alt=""><figcaption></figcaption></figure>

***

### <mark style="color:green;">Protected LSASS</mark>

In 2012, Microsoft implemented an LSA protection, to keep LSASS from being accessed to extract credentials from memory. To enable LSASS protection, we can modify the registry RunAsPPL DWORD value in `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa` to 1.

<figure><img src="../../../.gitbook/assets/image (169).png" alt=""><figcaption><p>Protection enabled</p></figcaption></figure>

Lucky for us, Mimikatz provides a mimidrv.sys driver that works on kernel level to disable the LSA protection. We can import it to Mimikatz by executing "!+" as follows,

```powershell
mimikatz # !+
```

<figure><img src="../../../.gitbook/assets/image (170).png" alt=""><figcaption><p>Importing driver</p></figcaption></figure>

Once the driver is loaded, we can disable the LSA protection by executing the following Mimikatz command:

```powershell
mimikatz # !processprotect /process:lsass.exe /remove
```

<figure><img src="../../../.gitbook/assets/image (171).png" alt=""><figcaption><p>Disable protection</p></figcaption></figure>

We can now dump the hashes:

```powershell
mimikatz # sekurlsa::logonpasswords
```

<figure><img src="../../../.gitbook/assets/image (172).png" alt=""><figcaption></figcaption></figure>
