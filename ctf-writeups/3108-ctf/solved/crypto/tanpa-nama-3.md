# Tanpa Nama 3

we are given cryptochalle.py

````python
```python
def xor_with_binary(binary_str, xor_str):
    binaries = binary_str.split()
    xor_num = int(xor_str, 2)
    xor_results = []
    for b in binaries:
        num = int(b, 2)
        result_num = num ^ xor_num
        xor_results.append(format(result_num, '08b'))
    return ' '.join(xor_results)

binary_str = "01010110 01010100 01010101 01011101 00011110 00110110 01010100 00101000 00110101 00101001 01010110 00111010 00100110 00110111 00110101 00111100 00110001 01010101 00111010 00100110 00101101 00100100 00101001 00101001 00100000 00101011 00100010 00100000 00011000"
xor_str = "01100101"
```
````

Again, this one doesn't have a print function so add one but before that, we have to call the function first because the XOR operation is not performed so just add&#x20;

<pre class="language-python"><code class="lang-python"><strong>result = xor_with_binary(binary_str, xor_str)
</strong>print(result)
</code></pre>

It will output a binary and just throw it into a decoder

<figure><img src="../../../../.gitbook/assets/image (445).png" alt=""><figcaption></figcaption></figure>

Flag: <mark style="color:red;">`3108{S1MPL3_CRPYT0_CHALLENGE}`</mark>&#x20;

