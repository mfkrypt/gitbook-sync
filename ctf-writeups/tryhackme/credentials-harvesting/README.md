# Credentials Harvesting

Credential harvesting consists of techniques for obtaining credentials like login information, account names, and passwords. It is a technique of extracting credential information from a system in various locations such as clear-text files, registry, memory dumping, etc.

As a red teamer, gaining access to legitimate credentials has benefits:

* It can give access to systems (Lateral Movement).
* It makes it harder to detect our actions.
* It provides the opportunity to create and manage accounts to help achieve the end goals of a red team engagement.

<figure><img src="../../../.gitbook/assets/image (178).png" alt=""><figcaption></figcaption></figure>

In this room, the focus will be on harvesting credentials from an internal perspective where a threat actor has already compromised a system and gained initial access.&#x20;

We have provided a Windows Server 2019 configured as a Domain Controller. To follow the content discussed in this room, deploy the machine and move on to the next task.

You can access the machine in-browser or through RDP using the credentials below.

Username: `thm`         Password: `Passw0rd!`&#x20;

<figure><img src="../../../.gitbook/assets/image (179).png" alt=""><figcaption></figcaption></figure>
