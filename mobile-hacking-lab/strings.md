# Strings



***

I will be using my own machine which I have setup with Android Studio and has Frida enabled. Launching the application displays a message

<figure><img src="../.gitbook/assets/image (840).png" alt=""><figcaption></figcaption></figure>

After unzipping the APK, I use `jadx-gui` to decompile source code. The first thing to look for is the AndroidManifest.xml file since it tells us how the app interacts with other system components

<figure><img src="../.gitbook/assets/image (839).png" alt=""><figcaption></figcaption></figure>

We can see the `intent-filter` tags being present with the VIEW action and BROWSABLE category.&#x20;

{% hint style="info" %}
In Android, Intents are messaging objects that facilitate communication between different app components (like activities, services, and broadcast receivers)
{% endhint %}

This setup allows it to be launched via a deep link with the scheme and host argument:

```
mhl://labs
```

We can try to launch `Activity2` by directly using the exported deep link using `adb`

```
❯ adb shell am start -d "mhl://labs" -n com.mobilehackinglab.challenge/.Activity2
```

Running this command will start the Intent

<figure><img src="../.gitbook/assets/image (842).png" alt=""><figcaption></figcaption></figure>

But it will close the app

<figure><img src="../.gitbook/assets/image (843).png" alt=""><figcaption></figcaption></figure>

Looking at the source code in `MainActivity` we can see two noticable parts:

<figure><img src="../.gitbook/assets/image (834).png" alt=""><figcaption></figcaption></figure>

The 1'st part attempts to load the library `libchallenge.so` .&#x20;

In the 2nd part, we can observe that the `KLOW()` method attempts to retrieve a `SharedPreferences` file named `DAD4` , sets up the date format, retrieves the current date and format it as a string.&#x20;

{% hint style="info" %}
A `SharedPreferences` object points to a file containing key-value pairs and provides simple methods to read and write them
{% endhint %}

And in this line, it stores the date string in the `SharedPreferences` file, `DAD4` under the key `UUUO133`

```java
editor.putString("UUU0133", cu_d);
```

Next, we will take a look at `Activity2` in the `getflag()`function

<figure><img src="../.gitbook/assets/image (831).png" alt=""><figcaption></figcaption></figure>

In the upper part of this huge block it is checking the `isActionView` and `isU1Matching` making sure the `u_1` variable which holds our `UUU0133` string matches the output of `cd()` method.&#x20;

Now, let's inspect the `cd()` method

<figure><img src="../.gitbook/assets/image (830).png" alt=""><figcaption></figcaption></figure>

Basically, it just retrieves the date format as a string and stores it in the shared variable `cu_d` .&#x20;

<figure><img src="../.gitbook/assets/image (832).png" alt=""><figcaption></figcaption></figure>

It hardcodes the secret key for the AES encryption:

```
your_secret_key_1234567890123456
```

It also hardcodes the string we want to decrypt which is:

```
bqGrDKdQ8zo26HflRsGvVA==
```

And in the multiple if blocks, it firsts checks the scheme and the host in the manifest file earlier and reads the last path segment of the URI and decodes it in base64. It compares it with `str` variable and if it passes it loads the Native library `lib.flag.so` and stores the flag in the `s` variable

To retrieve the correct base64 string to pass the checks, first we need to know the IV (Initialization Vector) . We can inspect the `decrypt()` method below

<figure><img src="../.gitbook/assets/image (824).png" alt=""><figcaption></figcaption></figure>

We can see it is retrieving the IV value from `Activity2Kt.fixedIV`

<figure><img src="../.gitbook/assets/image (825).png" alt=""><figcaption></figcaption></figure>

Great we now have the hardcoded IV value. To get the correct base64 string to validate the URI. We can use Cyberchef to:

1. Decode the hardcoded base64 string
2. AES decryption with hardcoded key and IV

<figure><img src="../.gitbook/assets/image (826).png" alt=""><figcaption></figcaption></figure>

And we get the string, `mhl_secret_1337` . Now, we encode this in base64 it becomes:

```
bWhsX3NlY3JldF8xMzM3
```

And now we can send the Intent similar to earlier but with `-d` for sending the data string with the correct URI using `adb`

```
❯ adb shell am start -a android.intent.action.VIEW -W -d "mhl://labs/bWhsX3NlY3JldF8xMzM3" -n com.mobilehackinglab.challenge/.Activity2
```

<figure><img src="../.gitbook/assets/image (844).png" alt=""><figcaption></figcaption></figure>

Let's check the Android emulator

<figure><img src="../.gitbook/assets/image (845).png" alt=""><figcaption></figcaption></figure>

We bypassed the checks! and it displays a new message. But the flag was nowhere to be found. Looking back at the description of the challenge, it guides us to scan the memory for the flag

<figure><img src="../.gitbook/assets/image (846).png" alt=""><figcaption></figcaption></figure>

I used fridump tool from this repo:

{% embed url="https://github.com/Nightbringer21/fridump" %}

It's the same as a memory scanner but it dumps the output. It takes relatively simple arguments. Only the running app name is needed. We can find this by running frida:

```
❯ frida-ps -U -a
```

<figure><img src="../.gitbook/assets/image (848).png" alt=""><figcaption></figcaption></figure>

And that is our app name. Now I used the fridump tool with `-U` option to specify it and the `-O` to output the dump files

```
python fridump.py -U Strings -o ~/mobile-hacking-lab/Strings
```

<figure><img src="../.gitbook/assets/image (849).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (850).png" alt=""><figcaption></figcaption></figure>

And that's done. Let's check the dump files

<figure><img src="../.gitbook/assets/image (851).png" alt=""><figcaption></figcaption></figure>

Yep, that's a lot. Following the hint, we can just grep out the flag

```
❯ strings *_dump.data | grep -i mhl
```

<figure><img src="../.gitbook/assets/image (852).png" alt=""><figcaption></figcaption></figure>

Nice challenge. LETSGOOO

flag: `MHL{IN_THE_MEMORY}`
