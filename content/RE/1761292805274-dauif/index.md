---
title: Write-up REEZS

---

Hello guys🤗! today i will write about a REV challenge that met at the CSCV2025 that named:***Reezs***.  

![image](https://hackmd.io/_uploads/SkxObFzAxe.png)

First when you install the file and extract the folder you will have an .exe file like this:     
![image](https://hackmd.io/_uploads/B1I12bfRel.png)
 When you run that file, you will see an CMD that told you enter the flag:
 ![image](https://hackmd.io/_uploads/r1JH3-MClx.png)
Since there were no clues at first, I typed some random input to see how the program executes. As expected, it shut down immediately.  
Then I opened IDA and loaded the file to read the main function and understand the execution flow. In IDA View-A, the main function looks like this:  

![image](https://hackmd.io/_uploads/SJ5kAWfCex.png)  

Let F5 to see that pseudocode:
```


int __fastcall main(int argc, const char **argv, const char **envp)
{
  __int64 v3; // rdx
  __int64 v4; // r8
  __m128i si128; // xmm0
  const char *v6; // rcx
  _BYTE v8[32]; // [rsp+0h] [rbp-58h] BYREF
  char Str[16]; // [rsp+20h] [rbp-38h] BYREF
  __m128i v10; // [rsp+30h] [rbp-28h]
  __int64 v11; // [rsp+40h] [rbp-18h]
  __int64 v12; // [rsp+48h] [rbp-10h]

  v10 = 0i64;
  *(_OWORD *)Str = 0i64;
  v11 = 0i64;
  sub_1400010F0("Enter flag: ", argv, envp);
  sub_140001170("%32s", Str);
  if ( strlen(Str) != 32 )
  {
    puts("No");
    if ( ((unsigned __int64)v8 ^ v12) == _security_cookie )
      return 0;
LABEL_5:
    __debugbreak();
  }
  si128 = _mm_load_si128((const __m128i *)&xmmword_14001E030);
  *(__m128i *)Str = _mm_xor_si128(_mm_load_si128((const __m128i *)Str), si128);
  v10 = _mm_xor_si128(si128, v10);
  if ( _mm_movemask_epi8(
         _mm_and_si128(
           _mm_cmpeq_epi8(*(__m128i *)Str, (__m128i)xmmword_140029000),
           _mm_cmpeq_epi8(v10, (__m128i)xmmword_140029010))) == 0xFFFF )
    v6 = (const char *)&unk_140023E7C;
  else
    v6 = "No";
  sub_1400010F0(v6, v3, v4);
  if ( ((unsigned __int64)v8 ^ v12) != _security_cookie )
    goto LABEL_5;
  return 0;
}

```    

What are the important parts of that code we should care about? Let me explain.
```

 sub_1400010F0("Enter flag: ", argv, envp);
  sub_140001170("%32s", Str);
```
This line ask us to type 32-byte strings input 32-byte strings(or we know the flag will have 32-byte strings not include the format flag)
```
 
si128 = _mm_load_si128((const __m128i *)&xmmword_14001E030);
```
This line point that it takes 16-byte constant of the **xmmord_14001E030** adress into **si128**
```

*(__m128i *)Str = _mm_xor_si128(_mm_load_si128((const __m128i *)Str), si128);
```
This line is load 16-byte of **Str** XOR with **si128** and the result is the first stage 16-byte strings of the **Str**
```

v10 = _mm_xor_si128(si128, v10);
```
**v10** at first is assigned to **0i64**, which is full of 0. If we XOR with **si128**, we will receive si128  

```
if ( _mm_movemask_epi8(
         _mm_and_si128(
           _mm_cmpeq_epi8(*(__m128i *)Str, (__m128i)xmmword_140029000),
           _mm_cmpeq_epi8(v10, (__m128i)xmmword_140029010))) == 0xFFFF )
```
This comparision can be analyze to smaller like this:
```

 _mm_cmpeq_epi8(*(__m128i *)Str, (__m128i)xmmword_140029000),
```
=>Return 16-byte, each byte = 0xFF if **(__m128i )Str == (__m128i)xmmword_140029000, else = 0x00**

```

_mm_and_si128(
```
=>Only keep if both comparisons are equal at the same position
```
_mm_movemask_epi8(
```
=>take top 16-bit of each byte,if the result is 0xFFFF it means the 16-bit verified(or all 16 bytes were equal in both comparisons).

If both of conditions are true, the program will print:
```
(const char *)&unk_140023E7C
```
That is yes:
```
.rdata:0000000140023E7C unk_140023E7C   db  59h ; Y             ; DATA XREF: main:loc_1400010CA↑o
.rdata:0000000140023E7D                 db  65h ; e
.rdata:0000000140023E7E                 db  73h ; s
.rdata:0000000140023E7F                 db    0
```


And i find this when point to **xmmword_140029010** and press X(jump to xref)

```
{
  BOOL result; // eax

  result = IsDebuggerPresent();
  if ( !result )
  {
    xmmword_140029010 = xmmword_14001E010;
    xmmword_140029000 = xmmword_14001E000;
  }
  return result;
}
```
=>If you run the binary in a debugger, those constants at **0x140029010** and **0x140029000** won’t be initialized=>an anti-debugger trick. And if you don't run in a debugger, 
**xmmword_140029010** will be **xmmword_14001E010** and   **xmmword_140029000** will be **xmmword_14001E000**

So i will use what i explained below to extract the flag:  

```
.rdata:000000014001E000 xmmword_14001E000 xmmword 939FCF9C9B9998C99DC8C9989ECFCB9Ah
.rdata:000000014001E010 xmmword_14001E010 xmmword 9F9D9D9DCB989A9B999A98CF9DCFCFCFh
.rdata:000000014001E030 xmmword_14001E030 xmmword 0AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAh
```
+The first stage we take first 16-bytes of **xmmword_14001E000** change into hex, then XOR with **xmmword_14001E030**

The the first 16-byte of **xmmword_14001E000** is:
```
unsigned char ida_chars[] = {
  0x9A, 0xCB, 0xCF, 0x9E, 0x98, 0xC9, 0xC8, 0x9D,
  0xC9, 0x98, 0x99, 0x9B, 0x9C, 0xCF, 0x9F, 0x93
};

```
XOR with 0xaa(**xmmword_14001E030**) into hex:
```
30 61 65 34 32 63 62 37 63 32 33 31 36 65 35 39

```
ASCII:
```
0ae42cb7c2316e59

```
+The second stage we take first 16-bytes of **xmmword_14001E010** change into hex, then XOR with **xmmword_14001E030**

The the first 16-byte of **xmmword_14001E000** is:
```
unsigned char ida_chars[] = { 0xCF, 0xCF, 0xCF, 0x9D, 0xCF, 0x98, 0x9A, 0x99, 0x9B, 0x9A, 0x98, 0xCB, 0x9D, 0x9D, 0x9D, 0x9F };

```
XOR with 0xaa(**xmmword_14001E030**) into hex:
```
65 65 65 37 65 32 30 33 31 30 32 61 37 37 37 35

```
ASCII:
```
eee7e203102a7775

```
And the flag is CSCV2025{0ae42cb7c2316e59eee7e203102a7775}

That's all of this challenge. Thanks for reading😘!