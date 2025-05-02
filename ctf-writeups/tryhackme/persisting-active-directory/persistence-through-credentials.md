# Persistence through Credentials

<figure><img src="../../../.gitbook/assets/image (658).png" alt=""><figcaption></figcaption></figure>

Together with your low-privileged credentials, you will be provided with Domain Administrator credentials. What luck! When discussing persistence techniques, you will use the privileged credentials to perform the persistence technique on your low-privileged credential set. Make a note of the following DA account:

Username: `Administrator`

Password: `tryhackmewouldnotguess1@`

Domain: `ZA`

***

### <mark style="color:green;">Not All Credentials Are Created Equal</mark>

We would usually search for privileged credentials such as those that are members of the Domain Admins group, these are also the credentials that will be rotated (a blue team term meaning to reset the account's password) first. If they did we would lose access

The goal is to persist with **near-privileged credentials**. We don't always need the full keys to the kingdom; we just need enough keys to ensure we can still achieve goal execution. We should attempt to persist through credentials such as the following:

* <mark style="color:red;">**Credentials that have local administrator rights on several machines**</mark>
* <mark style="color:red;">**Service accounts that have delegation permissions**</mark>
* <mark style="color:red;">**Accounts used for privileged AD services**</mark>: If we compromise accounts of privileged services such as Exchange, Windows Server Update Services (WSUS), or System Center Configuration Manager (SCCM), we could leverage AD exploitation to once again gain a privileged foothold.

***

### <mark style="color:green;">DCSync All</mark>

I previously explained about DCSync over here

{% embed url="https://mfkrypt.gitbook.io/stuff/ctf-writeups/tryhackme/exploiting-active-directory/exploiting-domain-trusts#dcsync" %}

But instead of one account, we want to DCSync every single account. To do this, we will have to enable logging on Mimikatz.

SSH into the DA account

<figure><img src="../../../.gitbook/assets/image (652).png" alt=""><figcaption></figcaption></figure>

Launch Mimikatz and enable logging

```powershell
mimikatz # log my_dcdump.txt 
Using 'my_dcdump.txt' for logfile: OK
```

DCSync with `/all` flag

```powershell
mimikatz # lsadump::dcsync /domain:za.tryhackme.loc /all
```

The file should be outputted and then download the file on the attacker machine. Now usually transferring this on to the attacker machine would be no problem but the file is so big its corrupted I can't transfer it without being corrupted. So I just used `secretsdump` to remotely dump all the hashes from the parent domain with the `-use-vss` flag

<figure><img src="../../../.gitbook/assets/image (656).png" alt=""><figcaption></figcaption></figure>

But that one was the parent domain, we want the hash of the krbtgt user on the child domain. This time I dumped without VSS

<figure><img src="../../../.gitbook/assets/image (657).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:green;">VSS</mark>

Volume Shadow Copy Service (**VSS**) is a **Windows feature** that creates **snapshots (backups) of files and volumes**, even when they are **in use**. It allows Windows to make **consistent backups** without interrupting running applications.
