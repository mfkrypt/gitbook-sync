# Ilmu Hisab

<figure><img src="../../../.gitbook/assets/image (581).png" alt=""><figcaption></figcaption></figure>

The challenge provides us with 2 netcat connections and a non stripped-ELF file

<figure><img src="../../../.gitbook/assets/image (582).png" alt=""><figcaption></figcaption></figure>

Inspecting the available functions, `main()` displays a message when connected to the netcat

<figure><img src="../../../.gitbook/assets/image (583).png" alt=""><figcaption></figcaption></figure>

`merdeka()` looks like the function that will display the flag

<figure><img src="../../../.gitbook/assets/image (584).png" alt=""><figcaption></figcaption></figure>

There is one interesting function called `addtwonumber()`, notably at this part

<figure><img src="../../../.gitbook/assets/image (586).png" alt=""><figcaption></figcaption></figure>

Let's rename the variables

<figure><img src="../../../.gitbook/assets/image (588).png" alt=""><figcaption></figcaption></figure>

`0x539` = 1337

`0x1ca3` = 7331

Basically we need to satisfy one of the two conditions then the we will get the flag. By looking at the first condition it looks illogical, and so is the second condition.

The solution here is to do an **Integer Overflow**.

In C, integers are stored in a fixed number of bits (32-bit, 64-bit). Each integer type has a maximum and minimum and maximum value it can hold. If the result of an operation exceeds this maximum or minimum, it **wraps around** to the opposite extreme, causing unexpected behavior.&#x20;

<figure><img src="../../../.gitbook/assets/image (589).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (592).png" alt=""><figcaption></figcaption></figure>

This also has a bit to do with signed and unsigned integers where

in a 32-bit system:

* <mark style="color:green;">Unsigned integer</mark>: Values from **0 to 4,294,967,295**
* <mark style="color:green;">Signed Integer</mark>: Values from **-2,147,483,648 to 2,147,483,647**

Anyways back to the challenge, if you look at the initial lines of the function, it was specfied that it uses `int` which is a 32-bit data type even though on a system that is 64-bit.

We can satisfy the first condition like this

<figure><img src="../../../.gitbook/assets/image (593).png" alt=""><figcaption></figcaption></figure>

The sum of the two numbers wrapped around to and became the negative part of the signed integer because of overflow.

Credits go to <mark style="color:red;">mhz,</mark> pinjam ss bang infra dah tutup hehe.
