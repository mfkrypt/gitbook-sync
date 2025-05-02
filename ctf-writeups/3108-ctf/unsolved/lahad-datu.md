---
description: >-
  Tetapi dokumen tersebut mempunyai kata laluan. Bantu Scott untuk membuka
  dokumen berkenaan.
---

# Lahad Datu

We are given a .docx file that is locked. The challenge description hints at a password that can be used

<figure><img src="../../../.gitbook/assets/image (533).png" alt=""><figcaption></figcaption></figure>

We can utilise **office2john** which is a tool that extracts hashes from Microsoft Office files then crack it using **John**

<figure><img src="../../../.gitbook/assets/office1 (1).png" alt=""><figcaption></figcaption></figure>

Note: **You will not find the password using the default John wordlist. Need to use rockyou wordlist**

<figure><img src="../../../.gitbook/assets/office2 (1).png" alt=""><figcaption></figcaption></figure>

And we got the password. Now open the document

<figure><img src="../../../.gitbook/assets/image (534).png" alt=""><figcaption></figcaption></figure>

Looks like we got the flag but its wrong when you try to submit it lol. The usual flags would actually mean something so I thought maybe it was encrypted and the bolded text was the key

Asked ChatGPT and bro said it was Vigenere Cipher so I just threw it into a decoder and got the flag

<figure><img src="../../../.gitbook/assets/image (535).png" alt=""><figcaption></figcaption></figure>

Flag: <mark style="color:red;">`3108{0P3R4S1_D4UL4T}`</mark>
