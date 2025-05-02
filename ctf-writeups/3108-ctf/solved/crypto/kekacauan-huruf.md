---
description: >-
  Balikkan proses penyulitan untuk mendedahkan bendera tersembunyi. Bolehkah
  anda menyahkod bendera dan menyelesaikan teka-teki? ~By GoogleTranslate :)
---

# Kekacauan Huruf

We are given chal.py and secret\_key.txt. For this challenge, I just let ChatGPT masak

solution script:

````python
```python
import random
from Crypto.Util.number import long_to_bytes

# Provided data from secret_key.txt
secret_key = [54, 38, 12, 47, 37, 37, 53, 22, 6, 38, 62, 22, 10, 54, 19, 41, 43, 53, 0, 62, 63, 28, 63, 63, 22, 10, 7, 37, 63, 53, 44, 8, 10, 42, 35, 43, 42, 63, 37, 21, 4, 19, 45, 21, 19, 18, 3, 62, 53, 24, 2, 62, 18, 35, 41, 14, 53, 3, 37, 63, 55, 62, 5]
offset = 50
padding_length = 9
original_order = [9, 20, 6, 12, 22, 38, 14, 24, 53, 52, 61, 29, 45, 11, 57, 44, 8, 46, 55, 59, 31, 2, 51, 43, 21, 27, 17, 40, 15, 58, 0, 26, 19, 36, 60, 28, 48, 39, 34, 50, 7, 16, 56, 30, 10, 49, 13, 3, 5, 42, 41, 47, 37, 4, 32, 33, 62, 1, 18, 23, 25, 35, 54]
q = 64

# Reverse the offset
recovered_secret_key = [(x - offset) % q for x in secret_key]

# Reverse the shuffling using original_order
original_secret_key = [0] * len(recovered_secret_key)
for index, shuffled_index in enumerate(original_order):
    original_secret_key[shuffled_index] = recovered_secret_key[index]

# Reconstruct the flag integer from the secret key
reconstructed_flag_int = 0
for value in reversed(original_secret_key):
    reconstructed_flag_int = reconstructed_flag_int * q + value

# Remove padding
reconstructed_flag_int >>= padding_length * 8

# Convert back to bytes
flag = long_to_bytes(reconstructed_flag_int)
print(flag)
```
````

Flag: <mark style="color:red;">`3108{9546880676d3788377699aad794c5a44}`</mark>
