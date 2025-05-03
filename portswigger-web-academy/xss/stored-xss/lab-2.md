# Lab 2

### Stored XSS into HTML context with nothing encoded

We are given a normal looking website, but there is a XSS vuln in the comment feature. Let's test with a simple alert

<figure><img src="../../../.gitbook/assets/image (106).png" alt=""><figcaption></figcaption></figure>

Looking at one of the posts available there is the comment functionality

<figure><img src="../../../.gitbook/assets/image (107).png" alt=""><figcaption></figcaption></figure>

Let's test it with a simple alert

```
<script>alert('pwned')</script>
```

<figure><img src="../../../.gitbook/assets/image (108).png" alt=""><figcaption></figcaption></figure>

Post the comment

<figure><img src="../../../.gitbook/assets/image (109).png" alt=""><figcaption></figcaption></figure>

And we solved the lab
