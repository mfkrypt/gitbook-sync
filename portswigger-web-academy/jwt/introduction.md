# Introduction

### JWT format

A JWT consists of 3 parts: a **header**, a **payload**, and a **signature**. These are each separated by a dot, as shown in the following example:

<figure><img src="../../.gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>

The header and payload parts of a JWT are just base64url-encoded JSON objects. For example, you can decode the payload from the token above to reveal the following claims:

```json
{ 
	"iss": "portswigger", 
	"exp": 1648037164, 
	"name": "Carlos Montoya", 
	"sub": "carlos", 
	"role": "blog_author", 
	"email": "carlos@carlos-montoya.net", 
	"iat": 1516239022 
}
```

***

### JWT Signature

The server that issues the token typically generates the signature by hashing the header and payload. In some cases, they also encrypt the resulting hash. Either way, this process involves a secret signing key.

* As the signature is directly derived from the rest of the token, changing a single byte of the header or payload results in a mismatched signature.
* Without knowing the server's secret signing key, it shouldn't be possible to generate the correct signature for a given header or payload.
