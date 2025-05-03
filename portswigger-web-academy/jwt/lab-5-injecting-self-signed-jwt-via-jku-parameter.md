# Lab 5 - Injecting Self Signed JWT via jku parameter

Some servers let you use the `jku` (JWK Set URL) header parameter to reference a JWK Set containing the key. When verifying the signature, the server fetches the relevant key from this URL.

**JWK Set**

A JWK Set is a JSON object containing an array of JWKs representing different keys. You can see an example of this below.

```json
{ 
	"keys": [ 
		{ 
			"kty": "RSA", 
			"e": "AQAB", 
			"kid": "75d0ef47-af89-47a9-9061-7c02a610d5ab", 
			"n": "o-yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9mk6GPM9gNN4Y_qTVX67WhsN3JvaFYw-fhvsWQ" 
		}, 
		{ 
		
			"kty": "RSA", 
			"e": "AQAB", 
			"kid": "d8fDFo-fS9-faS14a9-ASf99sa-7c1Ad5abA", 
			"n": "fc3f-yy1wpYmffgXBxhAUJzHql79gNNQ_cb33HocCuJolwDqmk6GPM4Y_qTVX67WhsN3JvaFYw-dfg6DH-asAScw" 
		} 
	] 
}
```

JWK Sets like this are sometimes exposed publicly via a standard endpoint, such as `/.well-known/jwks.json`

We log in to our account

<figure><img src="../../.gitbook/assets/image (73).png" alt=""><figcaption></figcaption></figure>

Grab the JWT Token from the session cookie

<figure><img src="../../.gitbook/assets/image (74).png" alt=""><figcaption></figcaption></figure>

This time also we will need to use the given exploit server to send our exploit

<figure><img src="../../.gitbook/assets/image (75).png" alt=""><figcaption></figcaption></figure>

To exploit this lab, we will use the trusty `jwt_tool` with the `-X` `s` to spoof the JWKS URL with the `-ju` to specify our given exploit server URL.

<figure><img src="../../.gitbook/assets/image (76).png" alt=""><figcaption></figcaption></figure>

```shell
python3 [TOKEN] -T -X s -ju [JWKSURL]
```

Now, we also need to tamper with it to make sure the `kid` header parameter matches the `kid` of the key

<figure><img src="../../.gitbook/assets/image (77).png" alt=""><figcaption></figcaption></figure>

\
Change the `sub` parameter to `administrator` and we get our generated Token. Before replacing the session cookie we need to send the generated JWKS in the exploit server

<figure><img src="../../.gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (79).png" alt=""><figcaption></figcaption></figure>

Store it here

<figure><img src="../../.gitbook/assets/image (80).png" alt=""><figcaption></figcaption></figure>

And we change the cookie we the generated Token and we should be able to access `/admin`

<figure><img src="../../.gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>
