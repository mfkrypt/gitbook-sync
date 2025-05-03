# Bypassing Weak Key Sanitization

nstead of the normal exploit:

```
/?__proto__.gadget=payload
```

If the sanitization process only strips the string `__proto__` only once and not recursively, an exploit like this:

```
/?__pro__proto__to__.gadget=payload
```

would easily bypass the restriction. Let's take a look at the lab
