---
description: >-
  Tapi ini bukan tentang kisah Kuching, ini kisah bagaimana ingin mendapatkan
  'flag' di dalam document yang berbahaya.
---

# Sarawak Kita

We are given a Microsoft maldoc

<figure><img src="../../../../.gitbook/assets/image (521).png" alt=""><figcaption></figcaption></figure>

Usually for these types I just use Oletools to analyze macros

<figure><img src="../../../../.gitbook/assets/image (523).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (524).png" alt=""><figcaption></figcaption></figure>

Looks like there are some sussy Base64 strings. Add `--decode` to print em

<figure><img src="../../../../.gitbook/assets/image (525).png" alt=""><figcaption></figcaption></figure>

Throw it into a decoder

<figure><img src="../../../../.gitbook/assets/image (526).png" alt=""><figcaption></figcaption></figure>

Flag: <mark style="color:red;">`3108{Kuch1ng_1bu_N3g3r1_S4r4w4k}`</mark>
