# Lab 6 - Injecting Self Signed JWT via kid parameter

Usually the `kid` in a JWT header tells the server which key to use to verify the token. Some servers look up this key in a **file** or a **database** using the `kid` value. If the server doesn't sanitize the `kid` , an attacker can use directory traversal in the `kid` like `../../../etc/passwd` treating its contents as the secret key.

One method to utilize this technique is pointing to a static file like `/dev/null` which returns an empty string. Thus, verifying the signature and the token!

If the server stores its verification keys in a database, the `kid` header parameter is also a potential vector for SQL injection attacks.

Now we login like usual with the given credentials

<figure><img src="../../.gitbook/assets/image (82).png" alt=""><figcaption></figcaption></figure>

Grab the JWT Token from the session cookie

<figure><img src="../../.gitbook/assets/image (83).png" alt=""><figcaption></figcaption></figure>

We need to sign it with its algorithm and the secret key which is an empty string because `/dev/null` will return an empty string. First we find the algorithm

<figure><img src="../../.gitbook/assets/image (84).png" alt=""><figcaption></figcaption></figure>

And now we use the `jwt_tool` by injecting claims that change the `kid` header to point to the `/dev/null` file and `sub` parameter to `administrator`

```shell
python3 jwt_tool.py [TOKEN] -I -hc kid -hv ../../../../dev/null -pc sub -pv administrator -S hs256 -p ''
```

<figure><img src="../../.gitbook/assets/image (85).png" alt=""><figcaption></figcaption></figure>

We get our Token. Just chuck it in and replace the cookie and we are able to access `/admin`

<figure><img src="../../.gitbook/assets/image (86).png" alt=""><figcaption></figcaption></figure>
