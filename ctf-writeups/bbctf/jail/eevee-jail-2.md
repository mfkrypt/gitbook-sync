# Eevee Jail - 2

This time we are given a simple Bash jail

{% tabs %}
{% tab title="jail-2.sh" %}
```sh
#!/bin/sh

echo "========================
=    Eevee's Jail 2    =
========================"

while :
do
	read -p "[+] > " huh
	o=`$huh`
done
```
{% endtab %}
{% endtabs %}

also a Docker setup file

{% tabs %}
{% tab title="docker-setup.txt" %}
1. sudo docker build --no-cache -t jail-3 .
2. sudo docker run --name eevee-jail-3 -p 9003:9003 --network custom-network -d jail-3
{% endtab %}
{% endtabs %}

Now this is a funny challenge with a LOT of attempts. There were no blacklisted inputs or anything at all. It only passes our inputs to `$huh` variable. It was a command substitution of some sort. I tried to directly read the flag and test some invalid shell commands:

```
❯ nc 157.180.92.15 9002

========================
=    Eevee's Jail 2    =
========================
[+] > cat flag.txt
cat flag.txt
...
...
[+] > lol
/jail-2/jail-2.sh: 1: lol: not found
[+] > 
```

So from here, we can observe that the valid command `cat flag.txt` is executing but not appearing in `stdout` . I also tried `bash` and `sh` . Funnily enough they work but we are still in the jail and unable to see the output.

```
[+] > bash
bash
bash: cannot set terminal process group (2907): Inappropriate ioctl for device
bash: no job control in this shell
eevee@ffe68f5882da:/jail-2$ cat flag.txt
cat flag.txt
eevee@ffe68f5882da:/jail-2$ 
```

I tried using the `*` command which tries to match all files in the current directory

```
[+] > *
*
/jail-2/jail-2.sh: 1: flag.txt: not found
```

This confirms that we are in the current directory and we just have to directly read `flag.txt` . After that, I tested the `.` command which is similar to `source` to execute the contents of `flag.txt`&#x20;

```
eevee@ffe68f5882da:/jail-2$ . flag.txt
. flag.txt
bash: bbctf{wow,: command not found
```

Its erring out and partially leaking the flag and this is where it gets funny if I tried this on the normal prompt it doesn't work lmaoo. I still dk why

```
[+] > . flag.txt
. flag.txt
/jail-2/jail-2.sh: 1: .: flag.txt: not found
[+] > 
```

Okay so the focus was on the shell prompt, its erring out so we can try to debug that using the `set -x` command and try `. flag.txt` again

```
eevee@ffe68f5882da:/jail-2$ set -x
set -x
eevee@ffe68f5882da:/jail-2$ . flag.txt
. flag.txt
+ . flag.txt
++ 'bbctf{wow,' thats 'interesting}'
bash: bbctf{wow,: command not found
eevee@ffe68f5882da:/jail-2$ 
```

And there's the flag

Flag: `bbctf{wow, thats interesting}`
