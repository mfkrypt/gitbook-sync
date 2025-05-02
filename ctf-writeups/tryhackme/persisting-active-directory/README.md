# Persisting Active Directory

Now that we have exploited AD and achieved some positions from which we can execute our goals, we need to make sure that we deploy persistence to make sure the blue team can't just kick us out. In this network, we will explore several different methods that could be used to persist in AD.

<figure><img src="../../../.gitbook/assets/image (650).png" alt=""><figcaption></figcaption></figure>

In this network, we will cover several methods that can be used to persist in AD. This is by no means a complete list, as available methods are usually highly situational and dependent on the AD structure and environment. However, we will cover the following techniques for persisting AD:

* AD Credentials and DCSync-ing
* Silver and Golden Tickets
* AD Certificates
* AD Security Identifiers (SIDs)
* Access Control Lists
* Group Policy Objects (GPOs)
