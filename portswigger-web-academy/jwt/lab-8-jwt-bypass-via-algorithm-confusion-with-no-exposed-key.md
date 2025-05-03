# Lab 8 - JWT bypass via algorithm confusion with no exposed key

If the public key is not available / exposed. We can use Portswigger's simple tool to get it for us using Docker

```
docker run --rm -it portswigger/sig2n <token1> <token2>
```

For each potential value, the script outputs:

* A Base64-encoded PEM key in both X.509 and PKCS1 format.
* A forged JWT signed using each of these keys.

To identify the correct PEM key to use. We need to test if the tokens when injected logs us out from our session or not. If that happens, then that is the wrong key

Let's try it out! Log into our usual account.

<figure><img src="../../.gitbook/assets/image (96).png" alt=""><figcaption></figcaption></figure>

Grab the 1'st JWT Token from the session

<figure><img src="../../.gitbook/assets/image (97).png" alt=""><figcaption></figcaption></figure>

Log out and log back in to get the 2nd JWT Token

<figure><img src="../../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>

Now we use the `sig2n` Docker tool to generate the Keys and Tokens

<figure><img src="../../.gitbook/assets/image (99).png" alt=""><figcaption></figcaption></figure>

Now, we test the Tokens.

<figure><img src="../../.gitbook/assets/image (100).png" alt=""><figcaption></figcaption></figure>

I tested the first token and I was not logged out. So, it is the correct Token. Now, we grab the Base64-encoded PEM key. Decode it and save it in a .pem file

<figure><img src="../../.gitbook/assets/image (102).png" alt=""><figcaption></figcaption></figure>

And after this I tried to solve the lab using the `jwt_tool` via the confusion attack method like previously shown but that didn't work. Apparently, we needed to use Burp to get the job done

{% embed url="https://youtu.be/4roTwhGSWZY?si=771P0CRaXKxywQzx" %}

Just followed the writeup and I got admin xD
