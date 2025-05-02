# Persistence through SID History

SIDs are used to track the security principal and the account's access when connecting to resources. There is, however, an interesting attribute on accounts called the SID history.

The legitimate use case of SID history is to enable access for an account to effectively be cloned to another. This becomes useful when an organisation is busy performing an AD migration as it allows users to retain access to the original domain while they are being migrated to the new one.

***

### <mark style="color:green;">History Can Be Whatever We Want It To Be</mark>

&#x20;With the right permissions, we can just add a SID of our current domain to the SID history of an account we control. Some interesting notes about this persistence technique:

* <mark style="color:red;">**Domain Admin privileges**</mark> (or equivalent) are usually required for this attack.
* When a user logs in, their <mark style="color:red;">**SIDs are added to their token**</mark>, defining their permissions.
* <mark style="color:red;">**Injecting the**</mark> <mark style="color:red;">**Enterprise Admin SID**</mark> grants **Domain Admin rights** across all domains in the forest.
* To stay even more hidden, the attacker can **modify the SID history** of another account.

***

### <mark style="color:green;">Forging History</mark>

Before we forge SID history, let's just SSH to the Administrator in the DC and get some information regarding the SIDs.

```powershell
Get-ADUser <your ad username> -properties sidhistory,memberof
```

<figure><img src="../../../.gitbook/assets/image (184).png" alt=""><figcaption></figcaption></figure>

We can confirm that our user does not currently have any SID History set. Let's get the SID of the Domain Admins group since this is the group we want to add to our SID History:

<figure><img src="../../../.gitbook/assets/image (185).png" alt=""><figcaption></figcaption></figure>

Now we add the SID history to our user but first we need to stop the stop the NTDS service and restart it back after done adding the SID history.

```powershell
Stop-Service -Name ntds -force
Add-ADDBSidHistory -SamAccountName 'username of our low-priveleged AD account' -SidHistory 'SID to add to SID History' -DatabasePath C:\Windows\NTDS\ntds.dit 
Start-Service -Name ntds
```

Then, we SSH to our low privileged user in THMWRK1 and check his SID history

<figure><img src="../../../.gitbook/assets/image (187).png" alt=""><figcaption></figcaption></figure>

Nice, we successfully added the SID of Domain Admins group to our low privileged user. We should be able to list directories and files of the DC

<figure><img src="../../../.gitbook/assets/image (188).png" alt=""><figcaption></figcaption></figure>
