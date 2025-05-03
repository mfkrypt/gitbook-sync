# Lab 6

### DOM XSS in jQuery anchor href attribute sink

This lab contains a DOM-based cross-site scripting vulnerability in the submit feedback page. It uses the jQuery library's `$` selector function to find an anchor element, and changes its `href` attribute using data from `location.search`.

To solve this lab, make the "back" link alert.

<figure><img src="../../../.gitbook/assets/image (126).png" alt=""><figcaption></figcaption></figure>

We can see there is a submit feedback function here. Let's inspect the script being loaded

<figure><img src="../../../.gitbook/assets/image (127).png" alt=""><figcaption></figcaption></figure>

In the script, jQuery's `attr()` function is used here to change the attributes of the anchor element `href` using data from the URL which is the source.

in this case the anchor is the back link's `href` , we can inject this payload in the URL

```
?returnUrl=javascript:alert('pwned')
```

After the malicious payload gets loaded into the `href` element, clicking the back link will execute the arbitrary code

<figure><img src="../../../.gitbook/assets/image (128).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (129).png" alt=""><figcaption></figcaption></figure>
