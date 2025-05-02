# AD Certificate Templates

I have already explained a bit about Certificates previously:

{% embed url="https://mfkrypt.gitbook.io/stuff/ctf-writeups/tryhackme/exploiting-active-directory/exploiting-certificates" %}

We can enumerate available Certificates Templates using `certutil`

```
certutil -v -template
```

We can also us [certipy](https://github.com/ly4k/Certipy) with the `-vulnerable` flag to enumerate vulnerable templates

```bash
certipy find -u <USER@DOMAIN> -p <PASSWORD> -dc-ip <IP> -vulnerable
```

<figure><img src="../../.gitbook/assets/image (160).png" alt=""><figcaption></figcaption></figure>

***

### <mark style="color:green;">Identifying Poisonous Combinations</mark>

For this task, there are three key parameters we should find, based on the dangerous combinations I mentioned previously. We are to find three parameters:

1. **Allow Enroll or Allow Full Control**
2. **Client Authentication**
3. _**CT\_FLAG\_ENROLLEE\_SUPPLIES\_SUBJECT**_

Looking at the output of the Certipy tool above, we find 2 templates that are vulnerable. The one we are looking at is `UserRequest`

<figure><img src="../../.gitbook/assets/image (163).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (162).png" alt=""><figcaption></figcaption></figure>

Great, now we have our vulnerable certificate!

