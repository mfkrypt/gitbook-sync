# fetch()

The `Fetch` API provides a simple way for developers to trigger HTTP requests using JavaScript. The `fetch()` method accepts two arguments:

* The URL to which you want to send the request.
* An **options object** that lets you to control parts of the request, such as the **method**, **headers**, **body parameters**, and so on.

The following is an example of how you might send a `POST` request using `fetch()`:

```js
fetch('https://normal-website.com/my-account/change-email', { 
	method: 'POST', 
	body: 'user=carlos&email=carlos%40ginandjuice.shop' 
})
```

As you can see, we've explicitly defined `method` and `body` properties, but there are a number of other possible properties that we've left undefined.

In this case, if an attacker can find a suitable source, they could potentially pollute `Object.prototype` with their own `headers` property. This may then be inherited by the options object passed into `fetch()` and subsequently used to generate the request.

This can lead to a number of issues. For example, the following code is potentially vulnerable to DOM XSS via prototype pollution:

```js
fetch('/my-products.json',{method:"GET"})
	.then((response) => response.json())
	.then((data) => {
	
		let username = data['x-username'];
		
		let message = document.querySelector('.message');
		
		if (username) {
			message.innerHTML = 'My products. Logged in as <b>${username}</b>`;
		}
		
		let productList = document.querySelector('ul.products');
		
		for(let product of data) {
			let product = document.createElement('li');
			product.append(product.name);
			productList.append(product);
		}
		
	})
	.catch(console.error);
```

To exploit this, an attacker could pollute `Object.prototype` with a `headers` property containing a malicious `x-username` header as follows:

```
?__proto__[headers][x-username]=<img/src/onerror=alert(1)>
```
