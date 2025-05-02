# Local Windows Credentials

### <mark style="color:green;">Security Account Manager (SAM)</mark>

The SAM is a Microsoft Windows database that contains local account information such as usernames and passwords. The SAM database stores these details in an encrypted format to make them harder to be retrieved. Moreover, it can not be read and accessed by any users while the Windows operating system is running. However, there are various ways and attacks to dump the content of the SAM database.

Located at C`:\Windows\System32\config\sam` it will look something like this when dumped:

```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:98d3b784d80d18385cea5ab3aa2a4261:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:ec44ddf5ae100b898e9edab74811430d:::
CREDS-HARVESTIN$:1008:aad3b435b51404eeaad3b435b51404ee:443e64439a4b7fe780db47fc06a3342d:::
```

***

### <mark style="color:green;">Volume Shadow Copy Service</mark>

I explained a bit about this previously

{% embed url="https://mfkrypt.gitbook.io/stuff/ctf-writeups/tryhackme/persisting-active-directory/persistence-through-credentials#vss" %}

With it, we can make a copy of the shadow volume and copy the SAM database

With cmd running as administrator, we will be using `wmic` to create a shadow volume copy from the C drive

```
wmic shadowcopy call create Volume='C:\'
```

Now, use `vssadmin` list and confirm that we have a shadow copy of the `C:` volume

```
vssadmin list shadows
```

<figure><img src="../../../.gitbook/assets/image (174).png" alt=""><figcaption></figcaption></figure>

The output shows that we have successfully created a shadow copy volume of (C:) with the following path: `\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1`

The SAM database is encrypted either with **RC4** or **AES** encryption algorithms. In order to decrypt it, we need a decryption key which is also stored in the files system in `c:\Windows\System32\Config\system`

In normal situation we would copy them from the Shadow copy and then transfer it onto our attacking machine and decrypt it

```
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\windows\system32\config\sam C:\users\Administrator\Desktop\sam
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\windows\system32\config\system C:\users\Administrator\Desktop\system
```

<figure><img src="../../../.gitbook/assets/image (175).png" alt=""><figcaption></figcaption></figure>

***

### <mark style="color:green;">Registry Hives</mark>

Another possible method for dumping the SAM database content is through the Windows Registry. Windows registry also stores a copy of some of the SAM database contents to be used by Windows services. Luckily, we can save the value of the Windows registry using the `reg.exe` tool. As previously mentioned, we need two files to decrypt the SAM database's content. Ensure you run the command prompt with Administrator privileges.

```
reg save HKLM\sam C:\users\Administrator\Desktop\sam-reg
reg save HKLM\system C:\users\Administrator\Desktop\system-reg
```

<figure><img src="../../../.gitbook/assets/image (176).png" alt=""><figcaption></figcaption></figure>

After downloading it on our attacking machine we can use impacket's `secretsdump` to dump the SAM hashes.

Note\*\*: This is not the same one, but it is using the same method

<figure><img src="../../../.gitbook/assets/image (177).png" alt=""><figcaption></figcaption></figure>
