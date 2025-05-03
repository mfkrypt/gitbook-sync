# JSON Spaces

The Express framework provides a `json spaces` option, which enables you to control the number of spaces used to indent any JSON data in the response.

```json
{
  "message": "Hello"
}
```

If it's set to `4`, the response might look like this:

```json
{
    "message": "Hello"
}
```

If you've got access to any kind of JSON response, you can try polluting the prototype with your own `json spaces` property, if `json spaces` is **not explicitly defined** in the Express application, we can issue the relevant request to confirm the vulnerability.

```json
{
  "__proto__": {
    "json spaces": 20
  }
}
```

* If the JSON response suddenly has **20 spaces of indentation**, it means the application is **vulnerable**.
* You can **reset it** by injecting `json spaces: 2` to restore the normal look.

**Note:**\
**When attempting this technique in Burp, remember to switch to the message editor's Raw tab in the Response. Otherwise, you won't be able to see the indentation change as the default prettified view normalizes this.**
