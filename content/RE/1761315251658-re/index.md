---
title: Write-up for Chatbot CSCV2025

---

Hello everyone💯! Today i will tell you the way to solve the ***Chatbot*** challenge in CSCV2025  

The question:
![image](https://hackmd.io/_uploads/r1KnQ9zRxx.png)

So at first you will have an file **main** like this below:
![image](https://hackmd.io/_uploads/HyCOV9zCgx.png)
Let open this file in DIE to see the information about this file:
![image](https://hackmd.io/_uploads/Sk8yH9zAll.png)
This is an ELF64 file and DIE has told us that this file have been packed with PyInstaller
Let use IDA(in app use SHIFT+F12) to see strings of this file:
![image](https://hackmd.io/_uploads/ByuAB9MRxx.png)
Scroll down a little bit and you will see this:
![image](https://hackmd.io/_uploads/rJ3ZLcf0xl.png)
We see many file like Py_ so that means this file was built by python
We will use **pyinstxtrator\.py** to export the content of this file(point in the **main** file to open terminal)
Type:
```
python pyinstxtractor.py main
```
Output will be:

![image](https://hackmd.io/_uploads/S126P9fCge.png)
Go back to the folder you put the **main** file and you will see there are an additional folder name **main_extracted**:
![image](https://hackmd.io/_uploads/Skoudcz0ll.png)

Now we will use a python decompiler to make main\.pyc file to main\.py file
I will use pylingual, link in here:https://www.pylingual.io/
The file after decompile to .py such like this:
```
# Decompiled with PyLingual (https://pylingual.io)
# Internal filename: main.py
# Bytecode version: 3.11a7e (3495)
# Source timestamp: 1970-01-01 00:00:00 UTC (0)

import base64
import json
import time
import random
import sys
import os
from ctypes import CDLL, c_char_p, c_int, c_void_p
from cryptography.hazmat.primitives import serialization, hashes
from cryptography.hazmat.primitives.asymmetric import padding
import ctypes

def get_resource_path(name):
    if getattr(sys, 'frozen', False):
        base = sys._MEIPASS
    else:  # inserted
        base = os.path.dirname(__file__)
    return os.path.join(base, name)

def load_native_lib(name):
    return CDLL(get_resource_path(name))
if sys.platform == 'win32':
    LIBNAME = 'libnative.dll'
else:  # inserted
    LIBNAME = 'libnative.so'
lib = None
check_integrity = None
decrypt_flag_file = None
free_mem = None
try:
    lib = load_native_lib(LIBNAME)
    check_integrity = lib.check_integrity
    check_integrity.argtypes = [c_char_p]
    check_integrity.restype = c_int
    decrypt_flag_file = lib.decrypt_flag_file
    decrypt_flag_file.argtypes = [c_char_p]
    decrypt_flag_file.restype = c_void_p
    free_mem = lib.free_mem
    free_mem.argtypes = [c_void_p]
    free_mem.restype = None
except Exception as e:
    print('Warning: native lib not loaded:', e)
    lib = None
    check_integrity = None
    decrypt_flag_file = None
    free_mem = None

def run_integrity_or_exit():
    if check_integrity:
        ok = check_integrity(sys.executable.encode())
        if not ok:
            print('[!] Integrity failed or debugger detected. Exiting.')
            sys.exit(1)
PUB_PEM = b'-----BEGIN PUBLIC KEY-----\nMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAsJftFGJC6RjAC54aMncA\nfjb2xXeRECiwHuz2wC6QynDd93/7XIrqTObeTpfBCSpOKRLhks6/nzZFTTsYdQCj\n4roXhWo5lFfH0OTL+164VoKnmUkQ9dppzpmV0Kpk5IQhEyuPYzJfFAlafcHdQvUo\nidkqcOPpR7hznJPEuRbPxJod34Bph/u9vePKcQQfe+/l/nn02nbfYWTuGtuEdpHq\nMkktl4WpB50/a5ZqYkW4z0zjFCY5LIPE7mpUNLrZnadBGIaLoVV2lZEBdLt6iLkV\nHXIr+xNA9ysE304T0JJ/DwM1OXb4yVrtawbFLBu9otOC+Gu0Set+8OjfQvJ+tlT/\nzQIDAQAB\n-----END PUBLIC KEY-----'
public_key = None
try:
    pub_path = get_resource_path('public.pem')
    if os.path.exists(pub_path):
        with open(pub_path, 'rb') as f:
            public_key = serialization.load_pem_public_key(f.read())
    else:  # inserted
        public_key = serialization.load_pem_public_key(PUB_PEM)
except Exception as e:
            print('Failed loading public key:', e)
            public_key = None

def b64url_encode(b):
    return base64.urlsafe_b64encode(b).rstrip(b'=').decode()

def b64url_decode(s):
    s = s | ('=', 4, len(s) - 4) | 4
    return base64.urlsafe_b64decode(s.encode())

def verify_token(token):
    if not public_key:
        return (False, 'no public key')
    try:
        payload_b64, sig_b64 = token.strip().split('.', 1)
        payload = b64url_decode(payload_b64)
        sig = b64url_decode(sig_b64)
        public_key.verify(sig, payload, padding.PKCS1v15(), hashes.SHA256())
        j = json.loads(payload.decode())
        if j.get('role')!= 'VIP':
            return (False, 'role != VIP')
        if j.get('expiry', 0) < int(time.time()):
            return (False, 'expired')
    else:  # inserted
        return (True, j)
    except Exception as e:
            return (False, str(e))

def sample_token_nonvip():
    payload = json.dumps({'user': 'guest', 'expiry': int(time.time()) + 3600, 'role': 'USER'}).encode()
    return b64url_encode(payload)

def main():
    run_integrity_or_exit()
    print('=== Bot Chat === \n    1.chat\n    2.showtoken\n    3.upgrade \n    4.quit')
    queries = 0
    while True:
        cmd = input('> ').strip().lower()
        if cmd in ['quit', 'exit']:
            return
        if cmd == 'chat':
            if queries < 3:
                print(random.choice(['Hi', 'Demo AI', 'Hello!', 'How can I assist you?', 'I am a chatbot', 'What do you want?', 'Tell me more', 'Interesting', 'Go on...', 'SIUUUUUUU', 'I LOVE U', 'HACK TO LEARN NOT LEARN TO HACK']))
                queries = queries | 1
            else:  # inserted
                print('Free queries exhausted. Use \'upgrade\'')
        else:  # inserted
            if cmd == 'showtoken':
                print('Token current:' + sample_token_nonvip())
            else:  # inserted
                if cmd == 'upgrade':
                    run_integrity_or_exit()
                    token = input('Paste token: ').strip()
                    ok, info = verify_token(token)
                    if ok:
                        if decrypt_flag_file is None:
                            print('Native library not available -> cannot decrypt')
                        else:  # inserted
                            flag_path = get_resource_path('flag.enc').encode()
                            res_ptr = decrypt_flag_file(flag_path)
                            if not res_ptr:
                                print('Native failed to decrypt or error')
                            else:  # inserted
                                flag_bytes = ctypes.string_at(res_ptr)
                                try:
                                    flag = flag_bytes.decode(errors='ignore')
                                except:
                                    flag = flag_bytes.decode('utf-8', errors='replace')
                                print('=== VIP VERIFIED ===')
                                print(flag)
                                free_mem(res_ptr)
                        return None
                    print('Token invalid:', info)
                else:  # inserted
                    print('Unknown. Use chat/showtoken/upgrade/quit')
if __name__ == '__main__':
    main()
```
Let analyze this python code:
```
def get_resource_path(name):
    if getattr(sys, 'frozen', False):
        base = sys._MEIPASS        # PyInstaller-style temp dir
    else:
        base = os.path.dirname(__file__)  # folder containing this .py
    return os.path.join(base, name)

    
```
That means all external files are searched next to the program
```
def load_native_lib(name):
    return CDLL(get_resource_path(name))
if sys.platform == 'win32':
    LIBNAME = 'libnative.dll'
else:  # inserted
    LIBNAME = 'libnative.so'
lib = None
check_integrity = None
decrypt_flag_file = None
free_mem = None
try:
    lib = load_native_lib(LIBNAME)
    check_integrity = lib.check_integrity
    check_integrity.argtypes = [c_char_p]
    check_integrity.restype = c_int
    decrypt_flag_file = lib.decrypt_flag_file
    decrypt_flag_file.argtypes = [c_char_p]
    decrypt_flag_file.restype = c_void_p
    free_mem = lib.free_mem
    free_mem.argtypes = [c_void_p]
    free_mem.restype = None
except Exception as e:
    print('Warning: native lib not loaded:', e)
    lib = None
    check_integrity = None
    decrypt_flag_file = None
    free_mem = None
```
This code tries to load a native library (libnative\.dll on Windows or libnative\.so on Linux) that exports:

**check_integrity(char) -> int
decrypt_flag_file(char) -> void
free_mem(void) -> None**

If loading fails, all three become None and the script prints a warning that native lib not loaded.

```
def run_integrity_or_exit():
    if check_integrity:
        ok = check_integrity(sys.executable.encode())
        if not ok:
            print('[!] Integrity failed or debugger detected. Exiting.')
            sys.exit(1)

```
"If" command show us **run_integrity_or_exit()** calls **check_integrity(sys.executable)**. If it returns 0, the program print **"Integrity failed or debugger detected. Exiting"** then exits.
```
PUB_PEM = b'-----BEGIN PUBLIC KEY-----\nMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAsJftFGJC6RjAC54aMncA\nfjb2xXeRECiwHuz2wC6QynDd93/7XIrqTObeTpfBCSpOKRLhks6/nzZFTTsYdQCj\n4roXhWo5lFfH0OTL+164VoKnmUkQ9dppzpmV0Kpk5IQhEyuPYzJfFAlafcHdQvUo\nidkqcOPpR7hznJPEuRbPxJod34Bph/u9vePKcQQfe+/l/nn02nbfYWTuGtuEdpHq\nMkktl4WpB50/a5ZqYkW4z0zjFCY5LIPE7mpUNLrZnadBGIaLoVV2lZEBdLt6iLkV\nHXIr+xNA9ysE304T0JJ/DwM1OXb4yVrtawbFLBu9otOC+Gu0Set+8OjfQvJ+tlT/\nzQIDAQAB\n-----END PUBLIC KEY-----'
public_key = None
try:
    pub_path = get_resource_path('public.pem')
    if os.path.exists(pub_path):
        with open(pub_path, 'rb') as f:
            public_key = serialization.load_pem_public_key(f.read())
    else:  # inserted
        public_key = serialization.load_pem_public_key(PUB_PEM)
except Exception as e:
            print('Failed loading public key:', e)
            public_key = None
```
It load an **RSA public key**: If a **public.pem** is next to the program it will use this file,else it turn back into **PUB_PEM**.
The next explaination will be copied from chatGPT(because i so lazy🥱):

**The “JWT” format used here (important!)**

This is not a standard 3-part JWT. The code expects exactly two parts:
```
token = "<payload_b64url>.<signature_b64url>"

```
+)The signed content is just the raw payload bytes (no header).

+)Signature algorithm: **RSA PKCS\#1 v1.5** with **SHA-256** (public_key.verify(sig, payload, PKCS1v15(), SHA256())).

+)The payload is JSON and must include:

**-"role": "VIP"
-"expiry": <unix_time_in_future>**

+)So a valid token = base64url(JSON) + “.” + base64url(RSA_signature_over_raw_payload).

Let go with our main job:Reverse!![image](https://hackmd.io/_uploads/r1AKa2GRxe.png)
So in this code if we can reverse the .so file we can find the decode flag function
To find the address of **decrypt_flag_file** we use:
```
nm -D libnative.so | grep -E 'decrypt_flag_file' 
```
The output is:
```

0000000000001610 T decrypt_flag_file

```
Let open **libnative\.so** in **radare2**
Because cannot determine entrypoint, **radare2** will use **0x00001220** at first
```
─$ r2 -w -e bin.cache=true -e bin.relocs.apply=true -A libnative.so
ERROR: Cannot determine entrypoint, using 0x00001220
INFO: Analyze all flags starting with sym. and entry0 (aa)
INFO: Analyze imports (af@@@i)
INFO: Analyze entrypoint (af@ entry0)
INFO: Analyze symbols (af@@@s)
INFO: Analyze all functions arguments/locals (afva@@@F)
INFO: Analyze function calls (aac)
INFO: Analyze len bytes of instructions for references (aar)
INFO: Finding and parsing C++ vtables (avrr)
INFO: Analyzing methods (af @@ method.*)
INFO: Recovering local variables (afva@@@F)
INFO: Type matching analysis for all functions (aaft)
INFO: Propagate noreturn information (aanr)
INFO: Use -AA or aaaa to perform additional experimental analysis
[0x00001220]> 

```
Let see the **disassenbly decrypt_flag_file** and the **pseudo-c** of this function
```
s 0x00001610(point to 0x1610)
af @ 0x00001610(define function)
pdf @ 0x00001610(print disassembly function)
pdc @ 0x00001610(decompile to pseudo-C)
```
The Output is:  
```
[0x00001220]> s 0x00001610
[0x00001610]> af @ 0x00001610
[0x00001610]> pdf @ 0x00001610
            ;-- rip:
┌ 579: sym.decrypt_flag_file (int64_t arg1);
│ `- args(rdi) vars(4:sp[0x10..0x54])
│           0x00001610      4157           push r15
│           0x00001612      4156           push r14
│           0x00001614      4155           push r13
│           0x00001616      4154           push r12
│           0x00001618      55             push rbp
│           0x00001619      53             push rbx
│           0x0000161a      4889fb         mov rbx, rdi                ; arg1
│           0x0000161d      4883ec28       sub rsp, 0x28
│           0x00001621      e8bafcffff     call sym.env_checks_ok
│           0x00001626      85c0           test eax, eax
│       ┌─< 0x00001628      0f84b2010000   je 0x17e0
│       │   0x0000162e      48c7442408..   mov qword [var_8h], 0
│       │   0x00001637      488d7c2408     lea rdi, [var_8h]
│       │   0x0000163c      e87ffaffff     call fcn.000010c0
│       │   0x00001641      4989c4         mov r12, rax
│       │   0x00001644      4885c0         test rax, rax
│      ┌──< 0x00001647      0f8493010000   je 0x17e0
│      ││   0x0000164d      48837c24080f   cmp qword [var_8h], 0xf
│     ┌───< 0x00001653      0f8677010000   jbe 0x17d0
│     │││   0x00001659      4889df         mov rdi, rbx                ; const char *filename                                                                                       
│     │││   0x0000165c      488d35c609..   lea rsi, [0x00002029]       ; "rb" ; const char *mode                                                                                    
│     │││   0x00001663      e828fbffff     call sym.imp.fopen          ; file*fopen(const char *filename, const char *mode)                                                         
│     │││   0x00001668      4889c3         mov rbx, rax
│     │││   0x0000166b      4885c0         test rax, rax
│    ┌────< 0x0000166e      0f845c010000   je 0x17d0
│    ││││   0x00001674      31f6           xor esi, esi                ; long offset
│    ││││   0x00001676      ba02000000     mov edx, 2                  ; int whence
│    ││││   0x0000167b      4889c7         mov rdi, rax                ; FILE *stream
│    ││││   0x0000167e      e8edfaffff     call sym.imp.fseek          ; int fseek(FILE *stream, long offset, int whence)                                                           
│    ││││   0x00001683      4889df         mov rdi, rbx                ; FILE *stream
│    ││││   0x00001686      e8a5f9ffff     call sym.imp.ftell          ; long ftell(FILE *stream)                                                                                   
│    ││││   0x0000168b      31d2           xor edx, edx                ; int whence
│    ││││   0x0000168d      31f6           xor esi, esi                ; long offset
│    ││││   0x0000168f      4889df         mov rdi, rbx                ; FILE *stream
│    ││││   0x00001692      4889c5         mov rbp, rax
│    ││││   0x00001695      e8d6faffff     call sym.imp.fseek          ; int fseek(FILE *stream, long offset, int whence)                                                           
│    ││││   0x0000169a      4883fd10       cmp rbp, 0x10
│   ┌─────< 0x0000169e      0f8e64010000   jle 0x1808
│   │││││   0x000016a4      4889ef         mov rdi, rbp                ; size_t size
│   │││││   0x000016a7      e804fbffff     call sym.imp.malloc         ;  void *malloc(size_t size)                                                                                 
│   │││││   0x000016ac      4989c5         mov r13, rax
│   │││││   0x000016af      4885c0         test rax, rax
│  ┌──────< 0x000016b2      0f8450010000   je 0x1808
│  ││││││   0x000016b8      4889d9         mov rcx, rbx                ; FILE *stream
│  ││││││   0x000016bb      4889ea         mov rdx, rbp                ; size_t nmemb
│  ││││││   0x000016be      be01000000     mov esi, 1                  ; size_t size
│  ││││││   0x000016c3      4889c7         mov rdi, rax                ; void *ptr
│  ││││││   0x000016c6      e8f5faffff     call sym.imp.fread          ; size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream)                                           
│  ││││││   0x000016cb      4889df         mov rdi, rbx                ; FILE *stream
│  ││││││   0x000016ce      4d8d7510       lea r14, [r13 + 0x10]
│  ││││││   0x000016d2      448d7df0       lea r15d, [var_10h]
│  ││││││   0x000016d6      e855faffff     call sym.imp.fclose         ; int fclose(FILE *stream)                                                                                   
│  ││││││   0x000016db      f3410f6f4500   movdqu xmm0, xmmword [r13]
│  ││││││   0x000016e1      0f29442410     movaps xmmword [var_sp_10h], xmm0
│  ││││││   0x000016e6      e805faffff     call sym.imp.EVP_CIPHER_CTX_new
│  ││││││   0x000016eb      4889c3         mov rbx, rax
│  ││││││   0x000016ee      4885c0         test rax, rax
│ ┌───────< 0x000016f1      0f8459010000   je 0x1850
│ │││││││   0x000016f7      48837c24081f   cmp qword [var_8h], 0x1f
│ ────────< 0x000016fd      0f86f5000000   jbe 0x17f8
│ │││││││   0x00001703      e8d8f9ffff     call sym.imp.EVP_aes_256_cbc
│ │││││││   0x00001708      4889c6         mov rsi, rax
│ │││││││   ; CODE XREF from sym.decrypt_flag_file @ 0x1800(x)
│ ────────> 0x0000170b      31d2           xor edx, edx
│ │││││││   0x0000170d      4c8d442410     lea r8, [var_sp_10h]
│ │││││││   0x00001712      4c89e1         mov rcx, r12
│ │││││││   0x00001715      4889df         mov rdi, rbx
│ │││││││   0x00001718      e833f9ffff     call sym.imp.EVP_DecryptInit_ex
│ │││││││   0x0000171d      85c0           test eax, eax
│ ────────< 0x0000171f      0f8423010000   je 0x1848
│ │││││││   0x00001725      4863fd         movsxd rdi, ebp             ; size_t size
│ │││││││   0x00001728      e883faffff     call sym.imp.malloc         ;  void *malloc(size_t size)                                                                                 
│ │││││││   0x0000172d      4889e2         mov rdx, rsp
│ │││││││   0x00001730      4589f8         mov r8d, r15d
│ │││││││   0x00001733      4c89f1         mov rcx, r14
│ │││││││   0x00001736      4889c6         mov rsi, rax
│ │││││││   0x00001739      4889df         mov rdi, rbx
│ │││││││   0x0000173c      c704240000..   mov dword [rsp], 0
│ │││││││   0x00001743      4889c5         mov rbp, rax
│ │││││││   0x00001746      c744240400..   mov dword [var_4h], 0
│ │││││││   0x0000174e      e87df9ffff     call sym.imp.EVP_DecryptUpdate
│ │││││││   0x00001753      85c0           test eax, eax
│ ────────< 0x00001755      0f84c5000000   je 0x1820
│ │││││││   0x0000175b      48633424       movsxd rsi, dword [rsp]
│ │││││││   0x0000175f      488d542404     lea rdx, [var_4h]
│ │││││││   0x00001764      4889df         mov rdi, rbx
│ │││││││   0x00001767      4801ee         add rsi, rbp
│ │││││││   0x0000176a      e801f9ffff     call sym.imp.EVP_DecryptFinal_ex
│ │││││││   0x0000176f      85c0           test eax, eax
│ ────────< 0x00001771      0f84a9000000   je 0x1820
│ │││││││   0x00001777      448b7c2404     mov r15d, dword [var_4h]
│ │││││││   0x0000177c      44033c24       add r15d, dword [rsp]
│ │││││││   0x00001780      418d7f01       lea edi, [r15 + 1]
│ │││││││   0x00001784      4863ff         movsxd rdi, edi             ; size_t size
│ │││││││   0x00001787      e824faffff     call sym.imp.malloc         ;  void *malloc(size_t size)                                                                                 
│ │││││││   0x0000178c      4989c6         mov r14, rax
│ │││││││   0x0000178f      4885c0         test rax, rax
│ ────────< 0x00001792      0f8488000000   je 0x1820
│ │││││││   0x00001798      4d63ff         movsxd r15, r15d
│ │││││││   0x0000179b      4889ee         mov rsi, rbp                ; const void *s2
│ │││││││   0x0000179e      4889c7         mov rdi, rax                ; void *s1
│ │││││││   0x000017a1      4c89fa         mov rdx, r15                ; size_t n
│ │││││││   0x000017a4      e857f9ffff     call sym.imp.memcpy         ; void *memcpy(void *s1, const void *s2, size_t n)                                                           
│ │││││││   0x000017a9      43c6043e00     mov byte [r14 + r15], 0
│ │││││││   0x000017ae      4889df         mov rdi, rbx
│ │││││││   0x000017b1      e89af9ffff     call sym.imp.EVP_CIPHER_CTX_free
│ │││││││   0x000017b6      4889ef         mov rdi, rbp                ; void *ptr
│ │││││││   0x000017b9      e8e2f9ffff     call sym.imp.free           ; void free(void *ptr)                                                                                       
│ │││││││   0x000017be      4c89ef         mov rdi, r13                ; void *ptr
│ │││││││   0x000017c1      e8daf9ffff     call sym.imp.free           ; void free(void *ptr)                                                                                       
│ │││││││   0x000017c6      4c89e7         mov rdi, r12                ; void *ptr
│ │││││││   0x000017c9      e8d2f9ffff     call sym.imp.free           ; void free(void *ptr)                                                                                       
│ ────────< 0x000017ce      eb13           jmp 0x17e3
│ │││││││   ; CODE XREFS from sym.decrypt_flag_file @ 0x1653(x), 0x166e(x)
│ │││└└───> 0x000017d0      4c89e7         mov rdi, r12                ; void *ptr
│ │││  ││   0x000017d3      e8c8f9ffff     call sym.imp.free           ; void free(void *ptr)                                                                                       
│ │││  ││   0x000017d8      0f1f840000..   nop dword [rax + rax]
│ │││  ││   ; CODE XREFS from sym.decrypt_flag_file @ 0x1628(x), 0x1647(x), 0x1818(x), 0x1840(x), 0x1860(x)                                                                         
│ ───┌┌└└─> 0x000017e0      4531f6         xor r14d, r14d
│ │││╎╎     ; CODE XREF from sym.decrypt_flag_file @ 0x17ce(x)
│ ────────> 0x000017e3      4883c428       add rsp, 0x28
│ │││╎╎     0x000017e7      4c89f0         mov rax, r14
│ │││╎╎     0x000017ea      5b             pop rbx
│ │││╎╎     0x000017eb      5d             pop rbp
│ │││╎╎     0x000017ec      415c           pop r12
│ │││╎╎     0x000017ee      415d           pop r13
│ │││╎╎     0x000017f0      415e           pop r14
│ │││╎╎     0x000017f2      415f           pop r15
│ │││╎╎     0x000017f4      c3             ret
..
│ │││╎╎     ; CODE XREF from sym.decrypt_flag_file @ 0x16fd(x)
│ ────────> 0x000017f8      e8b3f8ffff     call sym.imp.EVP_aes_128_cbc
│ │││╎╎     0x000017fd      4889c6         mov rsi, rax
│ ────────< 0x00001800      e906ffffff     jmp 0x170b
..
│ │││╎╎     ; CODE XREFS from sym.decrypt_flag_file @ 0x169e(x), 0x16b2(x)
│ │└└─────> 0x00001808      4889df         mov rdi, rbx                ; FILE *stream
│ │  ╎╎     0x0000180b      e820f9ffff     call sym.imp.fclose         ; int fclose(FILE *stream)                                                                                   
│ │  ╎╎     0x00001810      4c89e7         mov rdi, r12                ; void *ptr
│ │  ╎╎     0x00001813      e888f9ffff     call sym.imp.free           ; void free(void *ptr)                                                                                       
│ ────────< 0x00001818      ebc6           jmp 0x17e0
..
│ │  ╎╎     ; CODE XREFS from sym.decrypt_flag_file @ 0x1755(x), 0x1771(x), 0x1792(x)
│ ────────> 0x00001820      4889df         mov rdi, rbx
│ │  ╎╎     0x00001823      e828f9ffff     call sym.imp.EVP_CIPHER_CTX_free
│ │  ╎╎     0x00001828      4c89ef         mov rdi, r13                ; void *ptr
│ │  ╎╎     0x0000182b      e870f9ffff     call sym.imp.free           ; void free(void *ptr)                                                                                       
│ │  ╎╎     0x00001830      4c89e7         mov rdi, r12                ; void *ptr
│ │  ╎╎     0x00001833      e868f9ffff     call sym.imp.free           ; void free(void *ptr)                                                                                       
│ │  ╎╎     0x00001838      4889ef         mov rdi, rbp                ; void *ptr
│ │  ╎╎     0x0000183b      e860f9ffff     call sym.imp.free           ; void free(void *ptr)                                                                                       
│ │  └────< 0x00001840      eb9e           jmp 0x17e0
..
│ │   ╎     ; CODE XREF from sym.decrypt_flag_file @ 0x171f(x)
│ ────────> 0x00001848      4889df         mov rdi, rbx
│ │   ╎     0x0000184b      e800f9ffff     call sym.imp.EVP_CIPHER_CTX_free
│ │   ╎     ; CODE XREF from sym.decrypt_flag_file @ 0x16f1(x)
│ └───────> 0x00001850      4c89ef         mov rdi, r13                ; void *ptr
│     ╎     0x00001853      e848f9ffff     call sym.imp.free           ; void free(void *ptr)                                                                                       
│     ╎     0x00001858      4c89e7         mov rdi, r12                ; void *ptr
│     ╎     0x0000185b      e840f9ffff     call sym.imp.free           ; void free(void *ptr)                                                                                       
└     └───< 0x00001860      e97bffffff     jmp 0x17e0
[0x00001610]> pdc @ 0x00001610
 // callconv: rax amd64 (rdi, rsi, rdx, rcx, r8, r9, xmm0, xmm1, xmm2, xmm3, xmm4);
void sym.decrypt_flag_file (int64_t arg1) {
    loc_0x00001610:
        push (r15)
        push (r14)
        push (r13)
        push (r12)
        push (rbp)
        push (rbx)
        rbx = rdi     // arg1
        rsp -= 0x28   // (cstr 0x00000020) "@"
        sym.env_checks_ok ()
        v = eax & eax
        if (!v) goto loc_0x17e0 // likely
        goto loc_0x0000162e;
    loc_0x000017e0:
        // CODE XREFS from sym.decrypt_flag_file @ 0x1628(x), 0x1647(x), 0x1818(x), 0x1840(x), 0x1860(x)
        r14d = 0
    loc_0x000017e3:
        // CODE XREF from sym.decrypt_flag_file @ 0x17ce(x)
        rsp += 0x28
        rax = r14
        rbx = pop ()
        rbp = pop ()
        r12 = pop ()
        r13 = pop ()
        r14 = pop ()
        r15 = pop ()
        return
        goto loc_0x0000164d;
        return rax;
    loc_0x0000164d:
        v = qword [var_8h] - 0xf
        if (((unsigned) v) <= 0) goto 0x17d0 // likely
        goto loc_0x00001659;
    loc_0x000017d0:
        // CODE XREFS from sym.decrypt_flag_file @ 0x1653(x), 0x166e(x)
        rdi = r12     // void *ptr
        sym.imp.free ()  // void free(0)
         goto loc_0x000017e0
    loc_0x00001659:  // orphan
         rdi = rbx                // const char *filename
         rsi = rip + 0x9c6        // "rb" // 0x2029 // const char *mode
         sym.imp.fopen ()  // file*fopen(0, "rb")
         rbx = rax
         v = rax & rax
         if (!v) 
         goto loc_0x000017d0
    loc_0x00001674:  // orphan
         esi = 0                  // long offset
         edx = 2                  // int whence
         rdi = rax                // FILE *stream
         sym.imp.fseek () // int fseek(0, 0, 0x00000000)
         rdi = rbx                // FILE *stream
         sym.imp.ftell ()  // long ftell(0)
         edx = 0                  // int whence
         esi = 0                  // long offset
         rdi = rbx                // FILE *stream
         rbp = rax
         sym.imp.fseek ()  // int fseek(0, 0, 0)
         v = rbp - 0x10
         if (v <= 0) 
         goto loc_0x00001808
    loc_0x000016a4:  // orphan
         rdi = rbp                // size_t size // rsp
         sym.imp.malloc () //  void *malloc(0x00000000)
         r13 = rax
         v = rax & rax
         if (!v) 
         goto loc_0x00001808
    loc_0x000016b8:  // orphan
         rcx = rbx                // FILE *stream
         rdx = rbp                // size_t nmemb // rsp
         esi = 1                  // size_t size
         rdi = rax                // void *ptr
         sym.imp.fread () // size_t fread(0, 0x00000000, 0x00000000, 0)
         rdi = rbx                // FILE *stream
         r14 = r13 + 0x10
         r15d = var_10h
         sym.imp.fclose ()  // int fclose(0)
         xmm0 = xmmword [r13]
         xmmword [var_sp_10h] = xmm0
         sym.imp.EVP_CIPHER_CTX_new ()
         rbx = rax
         v = rax & rax
         if (!v) 
         goto loc_0x00001850
    loc_0x000016f7:  // orphan
         v = qword [var_8h] - 0x1f
         if (((unsigned) v) <= 0) 
         goto loc_0x000017f8
    loc_0x00001703:  // orphan
         sym.imp.EVP_aes_256_cbc ()
         rsi = rax
    loc_0x0000170b:  // orphan
         // CODE XREF from sym.decrypt_flag_file @ 0x1800(x)
         edx = 0
         r8 = var_sp_10h
         rcx = r12
         rdi = rbx
         sym.imp.EVP_DecryptInit_ex ()
         v = eax & eax
         if (!v) 
         goto loc_0x00001848
    loc_0x00001725:  // orphan
         rdi = ebp                // size_t size // rsp
         sym.imp.malloc () //  void *malloc(0x00000000)
         rdx = rsp
         r8d = r15d
         rcx = r14
         rsi = rax
         rdi = rbx
         dword [rsp] = 0
         rbp = rax
         dword [var_4h] = 0
         sym.imp.EVP_DecryptUpdate ()
         v = eax & eax
         if (!v) 
         goto loc_0x00001820
    loc_0x0000175b:  // orphan
         rsi = dword [rsp]
         rdx = var_4h
         rdi = rbx
         rsi += rbp               // rsp
         sym.imp.EVP_DecryptFinal_ex ()
         v = eax & eax
         if (!v) 
         goto loc_0x00001820
    loc_0x00001777:  // orphan
         r15d = dword [var_4h]
         r15d += dword [rsp]
         edi = r15 + 1
         rdi = edi                // size_t size
         sym.imp.malloc () //  void *malloc(0x00000000)
         r14 = rax
         v = rax & rax
         if (!v) 
         goto loc_0x00001820
    loc_0x00001798:  // orphan
         r15 = r15d
         rsi = rbp                // const void *s2 // rsp
         rdi = rax                // void *s1
         rdx = r15                // size_t n
         sym.imp.memcpy () // void *memcpy(0, 0x0000000000000000, 0)
         byte [r14 + r15] = 0
         rdi = rbx
         sym.imp.EVP_CIPHER_CTX_free ()
         rdi = rbp                // void *ptr // rsp
         sym.imp.free () // void free(0x0000000000000000)
         rdi = r13                // void *ptr
         sym.imp.free ()  // void free(0)
         rdi = r12                // void *ptr
         sym.imp.free ()  // void free(0)
         goto loc_0x000017e3
    loc_0x000017e3:  // orphan
         // CODE XREF from sym.decrypt_flag_file @ 0x17ce(x)
         rsp += 0x28
         rax = r14
         rbx = pop ()
         rbp = pop ()
         r12 = pop ()
         r13 = pop ()
         r14 = pop ()
         r15 = pop ()
         return
        return rax;
    loc_0x000017f8:  // orphan
         // CODE XREF from sym.decrypt_flag_file @ 0x16fd(x)
         sym.imp.EVP_aes_128_cbc ()
         rsi = rax
         goto loc_0x0000170b
    loc_0x00001808:  // orphan
         // CODE XREFS from sym.decrypt_flag_file @ 0x169e(x), 0x16b2(x)
         rdi = rbx                // FILE *stream
         sym.imp.fclose ()  // int fclose(0)
         rdi = r12                // void *ptr
         sym.imp.free ()  // void free(0)
         goto loc_0x000017e0
    loc_0x00001820:  // orphan
         // CODE XREFS from sym.decrypt_flag_file @ 0x1755(x), 0x1771(x), 0x1792(x)
         rdi = rbx
         sym.imp.EVP_CIPHER_CTX_free ()
         rdi = r13                // void *ptr
         sym.imp.free ()  // void free(0)
         rdi = r12                // void *ptr
         sym.imp.free ()  // void free(0)
         rdi = rbp                // void *ptr // rsp
         sym.imp.free () // void free(0x0000000000000000)
         goto loc_0x000017e0
    loc_0x00001848:  // orphan
         // CODE XREF from sym.decrypt_flag_file @ 0x171f(x)
         rdi = rbx
         sym.imp.EVP_CIPHER_CTX_free ()
    loc_0x00001850:  // orphan
         // CODE XREF from sym.decrypt_flag_file @ 0x16f1(x)
         rdi = r13                // void *ptr
         sym.imp.free ()  // void free(0)
         rdi = r12                // void *ptr
         sym.imp.free ()  // void free(0)
         goto loc_0x000017e0
}

```
We have dump all the flow of **decrypt_flag_file**,let patch **env_checks_ok** directly to return 1:
```
cp libnative.so libnative.so.bak(backup)
r2 -w -e bin.relocs.apply=true -q -c 's 0x000015a0; wx b801000000c3' libnative.so(patch)
r2 -e bin.relocs.apply=true -qc 'pd 3 @ 0x000015a0' libnative.so(verify)

```
The output is:
```
    ;-- env_checks_ok:
            0x000012e0      b801000000     mov eax, 1
            0x000012e5      c3             ret
            0x000012e6      0d0000488d     or eax, 0x8d480000

```
Let use this python code i take from chatGPT to bypass the chatbot entirely and call the native decryptor directly.
```
# bypass_vip_and_dump_flag_diag.py
import os, sys, ctypes
from ctypes import c_char_p, c_void_p, c_int

def get_resource_path(name):
    base = getattr(sys, "_MEIPASS", os.path.dirname(__file__))
    return os.path.join(base, name)

LIBNAME = "libnative.dll" if sys.platform == "win32" else "libnative.so"

lib_path  = os.path.abspath(get_resource_path(LIBNAME))
flag_path = os.path.abspath(get_resource_path("flag.enc"))
exe_path  = os.path.abspath(sys.executable)

print("[i] lib path :", lib_path)
print("[i] flag path:", flag_path)
print("[i] exe path :", exe_path)

if not os.path.exists(lib_path):
    raise SystemExit("[!] Library not found")
if not os.path.exists(flag_path):
    raise SystemExit("[!] flag.enc not found here")

lib = ctypes.CDLL(lib_path)

# prototypes (match the app)
try:
    check_integrity = lib.check_integrity
    check_integrity.argtypes = [c_char_p]
    check_integrity.restype  = c_int
except AttributeError:
    check_integrity = None

decrypt_flag_file = lib.decrypt_flag_file
decrypt_flag_file.argtypes = [c_char_p]
decrypt_flag_file.restype  = c_void_p

free_mem = lib.free_mem
free_mem.argtypes = [c_void_p]
free_mem.restype  = None

# 1) Some builds require this call first.
if check_integrity is not None:
    try:
        ok = check_integrity(exe_path.encode())
        print(f"[i] check_integrity({exe_path}) -> {ok}")
        # If it *must* pass, ok should be nonzero. If it returns 0, the lib may refuse to decrypt.
    except Exception as e:
        print("[!] check_integrity raised:", e)

# 2) Use ABSOLUTE path to flag.enc
ptr = decrypt_flag_file(flag_path.encode())

if not ptr:
    raise SystemExit("[!] decrypt_flag_file returned NULL (lib refused / error)")

try:
    flag_bytes = ctypes.string_at(ptr)
    try:
        flag = flag_bytes.decode()
    except UnicodeDecodeError:
        flag = flag_bytes.decode("utf-8", errors="replace")
    print("=== VIP VERIFIED ===")
    print(flag)
finally:
    free_mem(ptr)


```
The output when we run this python code should be:
```
└─$ python3 bypass_vip_and_dump_flag_diag.py                            
[i] lib path : /home/son/main_extracted/libnative.so
[i] flag path: /home/son/main_extracted/flag.enc
[i] exe path : /usr/bin/python3
[i] check_integrity(/usr/bin/python3) -> 1
=== VIP VERIFIED ===
CSCV2025{reversed_vip*_chatbot_bypassed}


```
So the flag is CSCV2025{reversed_vip*_chatbot_bypassed}. Thank for reading this post!💖💖💖