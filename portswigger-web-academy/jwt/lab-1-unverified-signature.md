# Lab 1 - Unverified Signature

JWT libraries typically provide one method for verifying tokens and another that just decodes them. For example, the Node.js library `jsonwebtoken` has `verify()` and `decode()`.

Occasionally, developers confuse these two methods and only pass incoming tokens to the `decode()` method. This effectively means that the application doesn't verify the signature at all.

### Objective: Gain access to `/admin`

We login to our account with the given credentials:`wiener:peter`

Inspecting the requests in Burp, we can see the JWT token here is within the our session cookie.

<figure><img src="../../.gitbook/assets/image (52).png" alt=""><figcaption></figcaption></figure>

Next, we willl use the `jwt_tool` from github to solve this. Copy the JWT Token ans use it in this command:

```bash
python3 jwt_tool.py [TOKEN] -T
```

Get to the payload part

<figure><img src="../../.gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>

We will try to modify `wiener` to `administrator`

<figure><img src="../../.gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>

Navigate to `/admin` and replace the cookie session

<figure><img src="../../.gitbook/assets/image (55).png" alt=""><figcaption></figcaption></figure>

Nice, it worked

<figure><img src="../../.gitbook/assets/image (56).png" alt=""><figcaption></figcaption></figure>
