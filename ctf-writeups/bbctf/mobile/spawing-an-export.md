---
description: >-
  This one was also a wasted challenge that I was overthinking but it was
  reallyyyyyyyyy easy
---

# Spawing An Export

Opening up the source code with `jadx-gui` . We check the MainActivity but nothing interestong was going on but there was a flag function in another section being defined.

<figure><img src="../../../.gitbook/assets/image (864).png" alt=""><figcaption></figcaption></figure>

Looking at the Manifest file we can observe the flag Activity here. The Activity just wasn't exported or "called"

<figure><img src="../../../.gitbook/assets/image (865).png" alt=""><figcaption></figcaption></figure>

We can use ActivityManager or `am` here with the `-n` flag to specify the component name of the activity we want to start

```
adb shell am start -n definitely.notvulnerable.spawn/.flag
```

<figure><img src="../../../.gitbook/assets/image (866).png" alt=""><figcaption></figcaption></figure>

Activity started. Now, we check the emulator

<figure><img src="../../../.gitbook/assets/image (867).png" alt=""><figcaption></figcaption></figure>

Flag: `bbctf{in$ECuRe_EXPORtED_aCt!vl71es}`
