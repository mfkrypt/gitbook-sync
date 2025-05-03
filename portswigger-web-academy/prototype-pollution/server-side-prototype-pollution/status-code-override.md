# Status Code Override

Server-side JavaScript frameworks like Express allow developers to set custom HTTP response statuses. Node's `http-errors` module contains the following function for generating this kind of error response:

```js
function createError () { 
	//... 
	if (type === 'object' && arg instanceof Error) {
		err = arg 
		status = err.status || err.statusCode || status
		
	} else if (type === 'number' && i === 0) { 
	//...
	 
	if (typeof status !== 'number' ||
	 
	(!statuses.message[status] && (status < 400 || status >= 600))) { 
		status = 500 
	}
	 
	//...
```

Now from this function, we can see from the first line, if an error is triggered it tries to read the `status` or `statusCode` property from the error object (`arg`) and tries to pass it to the `status` variable.

The status property of the error object **does not explicitly define** the `status` property. By now, we should already know undefined properties are vulnerable to Prototype Pollution

Besides that, in this block below the next condition is that the `status` property must be a number or between the range of `400-599` or else it will default to `500`

```js
if (typeof status !== 'number' ||
	 
	(!statuses.message[status] && (status < 400 || status >= 600))) { 
		status = 500 
	}
```

We can attempt to pollute the prototype using this:

```
Object.prototype.status = 418;
```
