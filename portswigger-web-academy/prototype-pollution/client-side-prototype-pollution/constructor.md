# Constructor

esides `__proto__` we can also use the `constructor` property

Unless an object's prototype is set to `null`, every Javascript object has a `constructor` property, which contains a reference to the constructor function that was used to create it. For example, you can create a new object either using literal syntax or by explicitly invoking the `Object()`

```js
let ObjectLiteral = {};   // Object literal
let ObjectSaya = new Object();  // Same thing but uses the Object constructor
```

We can reference the object using the `constructor` property:

```js
ObjectLiteral.constructor  // function Object(){...}
ObjectSaya.constructor  // function Object(){...}
```

As `ObjectSaya.constructor.prototype` is equivalent to `ObjectSaya.__proto__`, this provides an alternative vector for prototype pollution.
