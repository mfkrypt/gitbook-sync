---
description: >-
  Setiap negeri mempunyai daerah. Begitu juga negeri Sabah dan Sarawak mempunyai
  daerah tersendiri. Cari 'flag' yang mengandungi bilangan daerah Sabah dan
  Sarawak di dalam file tersebut.
---

# Daerah Sabah & Sarawak

We are given a zip file

<figure><img src="../../../../.gitbook/assets/image (448).png" alt=""><figcaption></figcaption></figure>

Extract it using binwalk, three jpg files are there

<figure><img src="../../../../.gitbook/assets/image (449).png" alt=""><figcaption></figcaption></figure>

Found out that "3.jpg" contains more files including another zip file named "BenderaKeNi.txt"

<figure><img src="../../../../.gitbook/assets/image (450).png" alt=""><figcaption></figcaption></figure>

The extracted files contain another .txt file which is a list of districts in Sabah and Sarawak

<figure><img src="../../../../.gitbook/assets/image (451).png" alt=""><figcaption></figcaption></figure>

"file.zip" is locked behind a password. First thought was Bruteforce with the available wordlist

<figure><img src="../../../../.gitbook/assets/image (452).png" alt=""><figcaption></figcaption></figure>

Got the password of the locked zip file, which was 'LubokAntu'

<figure><img src="../../../../.gitbook/assets/image (453).png" alt=""><figcaption></figcaption></figure>

Input the password when extracting the zip file

<figure><img src="../../../../.gitbook/assets/image (454).png" alt=""><figcaption></figcaption></figure>

Flag: <mark style="color:red;">`3108{S4B4H_27_D43RAH_S4R4W4K_40_D43R4H}`</mark>
