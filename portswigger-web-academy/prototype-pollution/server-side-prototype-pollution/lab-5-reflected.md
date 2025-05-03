# Lab 5 - Reflected

### Objective: Privilege Escalation

<figure><img src="../../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

\
We login using the given credentials:

`wiener:peter`

Looking at the fields below, looks like its updating the address. Let's take a look at the request being sent

<figure><img src="../../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

\
Yes, its goinng through the `/change-address` endpoint and sending a POST request with the data being in JSON.

<figure><img src="../../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

\
In the response section, we see a property set for our user `isAdmin: false`

<figure><img src="../../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

\
We can attempt to test Server-Side Prototype Pollution by polluting the global `Object.prototype`. First send the request to `Repeater` . Then, add our property

```json
{
	"__proto__":{
		"foo":"bar"
	}
}
```

Next send the request and look at the `Response`

<figure><img src="../../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

\
Success, we polluted the prototype. Now we can do stuff like change the `isAdmin` property to `true`

```json
{
	"__proto__":{
		"foo":"bar",
		"isAdmin":"true"
	}
}
```

Look at the `Response` again

<figure><img src="../../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

\
Success! Now refresh the website page and we have received Admin access
