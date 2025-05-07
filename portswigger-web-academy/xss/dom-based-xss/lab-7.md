# Lab 7

### DOM XSS in jQuery selector sink using a hashchange event

This lab contains a DOM-based cross-site scripting vulnerability on the home page. It uses jQuery's `$()` selector function to auto-scroll to a given post, whose title is passed via the `location.hash` property.

To solve the lab, deliver an exploit to the victim that calls the `print()` function in their browser.



<figure><img src="../../../.gitbook/assets/image (835).png" alt=""><figcaption></figcaption></figure>

We are given an exploit server this time to send the exploit

<figure><img src="../../../.gitbook/assets/image (836).png" alt=""><figcaption></figcaption></figure>

Inspecting the home page, there is indeed the `hashchange` event handler

<figure><img src="../../../.gitbook/assets/image (837).png" alt=""><figcaption></figcaption></figure>

```javascript
$(window).on('hashchange', function(){
    var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')');
    if (post) post.get(0).scrollIntoView();
});
                    
```

Using `window.location.hash` as the source, the hash is user-controllable and is injected directly into the jQuery selector. This makes the `$()` selector a **sink** — and a **vector for injection** — since the input is interpreted as part of a CSS selector, which can lead to **DOM-based XSS**

Now, we need to trigger a `hashchange` event. One of the simplest ways of doing this is to deliver the exploit via an `iframe`

```
<iframe src="https://example.com#" onload="this.src+='<img src=1 onerror=alert(1)>'">
```

In this example, the `src` attribute points to the vulnerable page with an empty hash value. When the `iframe` is loaded, an XSS vector is appended to the hash, causing the `hashchange` event to fire
