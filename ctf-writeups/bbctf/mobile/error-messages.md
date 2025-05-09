# Error Messages

We are given another apk file. I used `jadx-gui` to analyze the source code. Looking straight in MainActivity, we see a `getFlag()` function

<figure><img src="../../../.gitbook/assets/image (861).png" alt=""><figcaption></figcaption></figure>

After being defined, the function gets called and the string value gets logged. To get the flag we can use `Logcat` from the `adb` utility

I ran the apk on my emulator

<figure><img src="../../../.gitbook/assets/image (862).png" alt=""><figcaption></figcaption></figure>

And on my terminal, I ran `Logcat` and grepped the flag

```
❯ adb logcat | grep "bbctf"
```

<figure><img src="../../../.gitbook/assets/image (863).png" alt=""><figcaption></figcaption></figure>

Flag: `bbctf{V3rBo53_lO9G1n6_8a0bdbd84}`
