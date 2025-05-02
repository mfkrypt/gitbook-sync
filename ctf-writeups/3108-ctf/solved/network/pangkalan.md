---
description: >-
  Kami menerima transmisi dari pangkalan port pulau pinang bernombor 55663.
  Mohon segera untuk memberi bantuan.
---

# Pangkalan

We are given a pcap file "transmisi\_rahsia.pcap"

The challenge description hints about port 55663. So lets filter it accordingly for source OR destination.

<figure><img src="../../../../.gitbook/assets/image (518).png" alt=""><figcaption></figcaption></figure>

Now, if you look closely at the packets that contain \[PSH,ACK] flags, there are some lengths of data present

<figure><img src="../../../../.gitbook/assets/length.png" alt=""><figcaption></figcaption></figure>

So let's add another filter, that only leaves us with the packets that contain \[PSH,ACK] flags

<figure><img src="../../../../.gitbook/assets/image (519).png" alt=""><figcaption></figcaption></figure>

Another thing to notice is the data being transmitted is some characters with base64 padding

<figure><img src="../../../../.gitbook/assets/length2.png" alt=""><figcaption></figcaption></figure>

Just copy them at every packet and throw it into a decoder and we got a transmission

<figure><img src="../../../../.gitbook/assets/image (520).png" alt=""><figcaption></figcaption></figure>

The initials of the transmission is the flag

Flag: <mark style="color:red;">`3108{malbatt}`</mark>
