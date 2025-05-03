# Lab 1

### Reflected XSS into HTML context with nothing encoded

<figure><img src="../../../.gitbook/assets/image (103).png" alt=""><figcaption></figcaption></figure>

We are given a normal looking website, but there is a XSS vuln in the search feature. Let's test with a simple alert

```
<script>alert('pwned')</script>
```

<figure><img src="../../../.gitbook/assets/image (104).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (105).png" alt=""><figcaption></figcaption></figure>

Noice.
