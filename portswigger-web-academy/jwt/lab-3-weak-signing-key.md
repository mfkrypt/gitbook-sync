# Lab 3 - Weak Signing Key

Some signing algorithms, such as HS256 (HMAC + SHA-256), use an arbitrary, standalone string as the secret key resulting in the key can be easily guess/brute-forced

Login with the given credentials

<figure><img src="../../.gitbook/assets/image (62).png" alt=""><figcaption></figcaption></figure>

Grab the JWT Token from the session key

<figure><img src="../../.gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

We will use the `jwt_tool` to crack the secret key using the `-C` and `-d` for the dictionary

```shell
python3 jwt_tool.py [TOKEN] -C -d /usr/share/wordlists/rockyou.txt
```

<figure><img src="../../.gitbook/assets/image (64).png" alt=""><figcaption></figcaption></figure>

The secret key is: `secret1`

Now we change the `sub` parameter to `administrator` and sign the JWT Token with the secret key with `-S` to specify the algorithm and `-p` to specify the key

```shell
python3 jwt_tool.py [TOKEN] -T -S hs256 -p secret1
```

<figure><img src="../../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

Replace the session with the new generated Token and we can access `/admin`
