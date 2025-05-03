# Bypassing Input Filters

Similar to Client-Side Prototype Pollution, websites attempt to filter obvious keys like `__proto__` . This however, can be bypassed by:

* **Obfuscating** prohibited keywords
* Access the prototype via the **constructor property** instead of `__proto__`

```json
"constructor":{
	"prototype":{
		"foo":"bar"
	}
}
```

Node applications can also delete or disable `__proto__` altogether using the command-line flags `--disable-proto=delete` or `--disable-proto=throw` respectively. However, this can also be bypassed by using the constructor technique.
