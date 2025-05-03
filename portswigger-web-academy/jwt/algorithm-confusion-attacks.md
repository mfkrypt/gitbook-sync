# Algorithm Confusion Attacks

#### 🔄 What is Algorithm Confusion?

It’s when the server **gets confused** between different types of algorithms (like **RS256** vs **HS256**) because it **trusts the algorithm listed in the JWT header** — and blindly follows it.

#### ⚙️ Why Does It Happen?

Many JWT libraries use a **single `verify()` function** that:

* Reads the JWT's `alg` header (e.g., `"alg": "RS256"` or `"HS256"`)
* Chooses the verification method based on that
* Uses the **same key input**, no matter the algorithm

Example in **pseudo-code**:

```js
verify(token, key){
    if token.alg == "RS256":
        // Treat `key` as a public RSA key
    else if token.alg == "HS256":
        // Treat `key` as a symmetric secret key
}
```

💥 Where’s the Problem?

A developer might assume the server only accepts RS256 (which uses a public/private key pair), and they pass in:

```
verify(token, publicKey)
```

BUT, if the attacker changes the `alg` header to `HS256` it will automatically become a Symmetric-type authentication and it will accept the public key as the secret key. Thus, giving access.
