---
title: 'Write-up Square Cipher '

---

Hello guys! Because T1 was the champion of Worlds 2025 👩‍🦼‍➡️👩‍🦼‍➡️🥇🏆 so today i really happy to write about the challenge i have met in BuckeyeCTF 2025 named: ***Square Cipher***

# Question:
![image](https://hackmd.io/_uploads/HkozIule-x.png)
 
When we open the python file, we will get this code:
```
(lambda x=int(input('> '),16):(all((int(bin(y).translate(str.maketrans('1b','fx')),0)&x).bit_count()==15 for y in[511,261632,1838599,14708792,117670336,133955584,68585259008,35115652612096,246772580483072,1974180643864576,15793445150916608,17979214137393152,9205357638345293824,4713143110832790437888,4731607904558235517441,9463215809116471034882,18926431618232942069764,33121255085135066300416,37852863236465884139528,75705726472931768279056,151411452945863536558112,264970040681080530403328,302822905891727073116224,605645811783454146232448,1211291623566908292464896,2119760325448644243226624,2413129272746388704198656])and x&2135465562637171390290201561322170738230609084732268110734985633502584038857972308065155558608880==1271371190459412480076309932821732439054921890752535035282222258816851982409101952239053178406432 or print('incorrect'))and print(__import__('os').environ.get('FLAG','bctf{fake_flag}')))()

```
By my experience, this is an obfucated python code with the way to know is the amount of words in one line is so longgg!

So we need to deobfucate this code to see the real code
i deobfucated this code an get this
# True code
```
import os

def expand_mask(y: int) -> int:
   
    ys = [
        511, 261632, 1838599, 14708792, 117670336, 133955584,
        68585259008, 35115652612096, 246772580483072,
        1974180643864576, 15793445150916608, 17979214137393152,
        9205357638345293824, 4713143110832790437888, 4731607904558235517441,
        9463215809116471034882, 18926431618232942069764, 33121255085135066300416,
        37852863236465884139528, 75705726472931768279056, 151411452945863536558112,
        264970040681080530403328, 302822905891727073116224, 605645811783454146232448,
        1211291623566908292464896, 2119760325448644243226624, 2413129272746388704198656
    ]

    masks = [expand_mask(y) for y in ys]

    condA = all((mask & x).bit_count() == 15 for mask in masks)

    BIG_CONST = 2135465562637171390290201561322170738230609084732268110734985633502584038857972308065155558608880
    REQUIRED = 1271371190459412480076309932821732439054921890752535035282222258816851982409101952239053178406432
    condB = (x & BIG_CONST) == REQUIRED

    return condA and condB

def main():
    x = int(input('> '), 16)

    if not check_x(x):
        print('incorrect')
        return
    print(os.environ.get('FLAG', 'bctf{fake_flag}'))

if __name__ == '__main__':
    main()

```
#### Let analyse this code:
1. It takes hex value from user  
```
x = int(input('> '), 16)
```
2. It builds a list of masks by taking the binary representation of each integer y, replacing every '1' with 'f' and every 'b' with 'x', producing strings and then parses those as hex
3. For every mask, x must have exactly 15 bits set inside the positions that mask selects
```
 condA = all((mask & x).bit_count() == 15 for mask in masks)
```
4. It requires a bitwise equality
```
(x & BIG_CONST) == REQUIRED
```
5.If check fail,reponse incorrect.If it true, return back bctf{fake_flag} (in this local code) or the true flag in the sever code
# Solve
i use chatGPT to solve the key for me and i get this:
```
Nice — I solved it. The hex input that passes the checks is:

6986853c14b89c434c5a43528726075d0e3068c4c6338a3105e46514bb188178a942628f538587621
```
Run the code below and paste this hex so i get the local flag:
![image](https://hackmd.io/_uploads/HkIvDtglZg.png)
That is the corrext hex!, let run the netcat server and use this hex to get the real flag:
![image](https://hackmd.io/_uploads/Hkg8uYeeZg.png)

Flag:bctf{5um_0f_f1r57_n_0dd_numb3r5_c1ph3r_025165aa}
Thanks for watching this post!🌸🌸🌸🌸🌸
