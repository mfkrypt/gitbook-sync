# Lab 2 - Flawed Signature Verification

Among other things, the JWT header contains an `alg` parameter. JWTs can be signed using a range of different algorithms, but can also be left unsigned. In this case, the `alg` parameter is set to `none`, which indicates a so-called "unsecured JWT".

Log in normally with the given credentials

<figure><img src="../../.gitbook/assets/image (57).png" alt=""><figcaption></figcaption></figure>

The JWT Token is in the session cookie

<figure><img src="../../.gitbook/assets/image (58).png" alt=""><figcaption></figcaption></figure>

We use the tool, `jwt_tool` to modify `wiener` to `administrator` and change the `alg` parameter value from `RS256` to `none`

```shell
python3 jwt_tool.py [TOKEN] -T
```

<figure><img src="../../.gitbook/assets/image (59).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>

Use the tampered JWT Token and replace the cookie but we only take the `header` and `payload` part . The signature is removed (leaving the trailing period)

```
eyJraWQiOiJhMWNjODc1OS02NGEwLTRhZGItYTczMy1hZjdhOTM0ZTcyZDkiLCJhbGciOiJub25lIn0.eyJpc3MiOiJw
b3J0c3dpZ2dlciIsImV4cCI6MTc0NTA1Njk1OCwic3ViIjoiYWRtaW5pc3RyYXRvciJ9.
```

<figure><img src="../../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>
