# Definition

### What is it?

Prototype pollution is an injection attack that targets JavaScript runtimes that enables an attacker to add arbitrary properties to global object prototypes, which may then be inherited by user-defined objects.

With prototype pollution, an attacker might control the default values of an object's properties to get RCE or perform DOS attacks.

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

***

### How vulnerabilities arise?

Prototype pollution happens when a JavaScript function merges user-controlled input into an existing object without checking the keys. An attacker can add a key like `__proto__`, which modifies the object's prototype instead of just the object itself. This can inject harmful properties into all objects that inherit from it, potentially leading to security issues.

It's possible to pollute any prototype object, but this most commonly occurs with the built-in global `Object.prototype`.
