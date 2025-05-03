# Lab 7 - JWT bypass via algorithm confusion

If the application uses JWTs with public key based signatures, but does not check that the algorithm is correct, this can potentially exploited in a signature type confusion attack.

1. The application must expect the JWT to be signed with a public key based algorithm (i.e, `RSxxx` or `ESxxx`).
2. The application must not check which algorithm the JWT is actually using for the signature.
3. The public key used to verify the JWT must be available to the attacker.

If all of these conditions are true, then an attacker can use the public key to sign the JWT using a HMAC based algorithm (such as `HS256`).

**This means that if the JWT is signed using `publicKey` as a secret key for the `HS256` algorithm, the signature will be considered valid.**

Let's check out the lab now. Login with usual credentials

<figure><img src="../../.gitbook/assets/image (87).png" alt=""><figcaption></figcaption></figure>

Grab the JWT Token from the session cookie

<figure><img src="../../.gitbook/assets/image (88).png" alt=""><figcaption></figcaption></figure>

As per the requirements for the confusion attack. We need an available public key that is used for signing the Token. Let's check the default endpoint which is `/jwks.json`

<figure><img src="../../.gitbook/assets/image (89).png" alt=""><figcaption></figcaption></figure>

And yes, it's present. Now this is in JWK format we need to convert it into a PEM format as a public key file format. We can use an online tools for this

{% embed url="https://8gwifi.org/jwkconvertfunctions.jsp" %}

<figure><img src="../../.gitbook/assets/image (91).png" alt=""><figcaption></figcaption></figure>

Now grab that and save it in a file. I will call it `pk.pem`

<figure><img src="../../.gitbook/assets/image (92).png" alt=""><figcaption></figcaption></figure>

Let us now use the `jwt_tool` with the `-X k` option to specify the confusion exploit

<figure><img src="../../.gitbook/assets/image (93).png" alt=""><figcaption></figcaption></figure>

We also need to use the `-pk` to specify the public key PEM file earlier and change the `sub` to `administrator`

```
python3 jwt_tool.py [TOKEN] -X k -pk pk.pem -I -pc sub -pv administrator
```

<figure><img src="../../.gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>

And just change the new Token in the session cookie and we get `admin`!

<figure><img src="../../.gitbook/assets/image (95).png" alt=""><figcaption></figcaption></figure>
