# Prototypes & Inheritance

### What is a Prototype?

In JavaScript, every object has a **prototype**, which is like a **blueprint** for the object. By default, JavaScript automatically assigns new objects one of its built-in prototypes. For example, strings are automatically assigned the built-in `String.prototype`.

```javascript
let myObject = {};
Object.getPrototypeOf(myObject);  // Object.prototype 

let myString = "";
Object.getPrototypeOf(myString);  // String.prototype

let myArray = [];
Object.getPrototypeOf(myArray);  // Array.prototype

let myNumber = 1;
Object.getPrototypeOf(myNumber);  // Number.prototype

```

***

### How does object inheritance work?

Whenever you reference a property of an object, the JavaScript engine first tries to access this directly on the object itself. If the object doesn't have a matching property, the JavaScript engine looks for it on the object's prototype instead. Given the following objects, this enables you to reference `myObject.propertyA`, for example:

<figure><img src="../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

We can also see this behaviour in the browser console:

```javascript
let myObject = {};
```

Next, type `myObject` followed by a dot. Notice that the console prompts you to select from a list of properties and methods:

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Even though there are no properties or methods defined for the object itself, it has inherited some from the built-in `Object.prototype`.

***

### The Prototype Chain

Note that an object's prototype is just another object, which should also have its own prototype, and so on. As virtually everything in JavaScript is an object under the hood, this chain ultimately leads back to the top-level `Object.prototype`, whose prototype is simply `null`.

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

In the example above, this means that the `username` object has access to the properties and methods of both `String.prototype` and `Object.prototype`

***

### Accessing an object's prototype using `__proto__`

Most browsers use `__proto__` property to access the prototype of an object. This property serves as both a getter and setter for the object's prototype. This means you can use it to read the prototype and its properties, and even reassign them if necessary.

We can access `__proto__` using either bracket or dot notation:

```javascript
username.__proto__
username['__proto__']
```

We can even chain references to `__proto` and work our way up to the prototype chain:

```javascript
username.__proto__                            // String.prototype
username.__proto__.__proto__                  // Object.prototype
username.___proto__.__proto__.__proto__       // NULL
```

***

### Modifying Prototypes

For example, modern JavaScript provides the `trim()` method for strings, which enables you to easily remove any leading or trailing whitespace. Before this built-in method was introduced, developers sometimes added their own custom implementation of this behavior to the `String.prototype` object by doing something like this:

```javascript
String.prototype.removeWhitespace = function() {
	// Remove leading and trailing whitespace
}
```

Thanks to prototypal inheritance, all strings would then have access to this method:

```javascript
let searchTerm = '  example ';
searchTerm.removeWhitespace();
```
