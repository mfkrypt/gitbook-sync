# Lab 3

### DOM XSS in document.write sink using source location.search

A website is vulnerable to DOM-based XSS if data flows from a source to a sink in an executable path. In practice, different sources and sinks behave differently, affecting how an exploit works. Scripts may also validate or modify data, which needs to be accounted for during exploitation.

This lab contains a DOM-based cross-site scripting vulnerability in the search query tracking functionality. It uses the JavaScript `document.write` function, which writes data out to the page. The `document.write` function is called with data from `location.search`, which you can control using the website URL.

To solve this lab, perform a cross-site scripting attack that calls the `alert` function.

<figure><img src="../../../.gitbook/assets/image (110).png" alt=""><figcaption></figcaption></figure>

So I tried a simple alert

```
<script>alert('pwned')</script>
```

<figure><img src="../../../.gitbook/assets/image (111).png" alt=""><figcaption></figcaption></figure>

But.. that did not work. So I inspected the element on how it was injected.

<figure><img src="../../../.gitbook/assets/image (112).png" alt=""><figcaption></figcaption></figure>

It looks like it is enclosed in an `img src` attribute. We can escape out of the `img` attribute using this:

```
"><svg onload=alert('pwned')>
```

<figure><img src="../../../.gitbook/assets/image (113).png" alt=""><figcaption></figcaption></figure>
