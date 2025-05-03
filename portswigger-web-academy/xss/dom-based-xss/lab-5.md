# Lab 5

### DOM XSS in innerHTML

<figure><img src="../../../.gitbook/assets/image (124).png" alt=""><figcaption></figcaption></figure>

There is a DOM XSS vulnerability in the search feature.

It uses an `innerHTML` assignment. The `innerHTML` sink doesn't accept `script` elements on any modern browser, nor will `svg onload` events fire.

We can counter this by using this payload:

```
<img src=1 onerror=alert('pwned')>
```

Which loads an image from an invalid source and then calls the `alert` function when an error is triggered

<figure><img src="../../../.gitbook/assets/image (125).png" alt=""><figcaption></figcaption></figure>
