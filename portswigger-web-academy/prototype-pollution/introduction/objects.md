# Objects

### What is an Object?

A JavaScript object is essentially just a collection of `key:value` pairs known as "properties". For example, the following object could represent a user:

```javascript
const user = { 
	username: "wiener", 
	userId: 01234, 
	isAdmin: false
}
```

You can access the properties of an object by using either dot notation or bracket notation to refer to their respective keys:

```javascript
user.username // "wiener" 
user['userId'] // 01234
```

Properties may also contain functions known as a **method**

```javascript
const user = {
	username: "wiener",
	userID: 01234,
	ContohMethod: function(){
		// do Something
	}
}
```
