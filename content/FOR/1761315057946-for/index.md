---
title: 'CSCV2025 FOR DNS '

---

FORENSICS CSCV 2025 WRITE UP
From: MTA.Support

FOR1: DNS


After downloading and unzipping the file, we will get the following 3 files
- capture.pcap
- access.log
- error.log

First open the .pcap file with wireshark and get the following interface

![image](https://hackmd.io/_uploads/B1GY8IfAeg.png)

Rearrange the section information and scroll from top to bottom 

We find the suspicious dns with information ending in hex.cloudflar3.com


![image](https://hackmd.io/_uploads/S1riwLMRlx.png)
![image](https://hackmd.io/_uploads/H1l6wLf0le.png)

Get the query response, we has:

p.3dfec8a22cde4db4463db2c35742062a415441f526daecb59b.hex.cloudflar3.com
p.c7aec5d0d81ba8748acac6931e5add6c24b635181443d0b9d2.hex.cloudflar3.com
p.f6af1ecb8cc9827a259401e850e5e07fdc3c1137f1.hex.cloudflar3.com
p.f8aad90d5fc7774c1e7ee451e755831cd02bfaac3204aed8a4.hex.cloudflar3.com
f.6837abc6655c12c454abe0ca85a596e98473172829581235dd.hex.cloudflar3.com f.95380b06bf6dd06b89118b0003ea044700a5f2c4c106c3.hex.cloudflar3.com

Concatenate the hex segments in the query

c7aec5d0d81ba8748acac6931e5add6c24b635181443d0b9d2f8aad90d5fc7774c1e7ee451e755831cd02bfaac3204aed8a43dfec8a22cde4db4463db2c35742062a415441f526daecb59bf6af1ecb8cc9827a259401e850e5e07fdc3c1137f16837abc6655c12c454abe0ca85a596e98473172829581235dd95380b06bf6dd06b89118b0003ea044700a5f2c4c106c3

Navigate to file error.log, we try to find some special strings such as “CSCV, FLAG, FORENSIC, ..”

Fortunately, we found this line contain some suspicious data 

![image](https://hackmd.io/_uploads/SkPwGCQAge.png)


From the image, we get a lot of important information from just 2 lines

APP_SECRET=F0r3ns1c-2025-CSCV
H=SHA256(APP_SECRET)
AES_KEY=H[0.15]
AES_IV=H[16.31]

So that, we can be sure that the flag was decrypted by AES CBC with the key and iv in the detail of 2 lines

Since, we have the sha256sum of APP_SECRET is: 5769179ccdf950443501d9978f52ddb51b70ca0d4f607a976c6639914af7c7a6

=> KEY = 5769179ccdf950443501d9978f52ddb5
=> IV = 1b70ca0d4f607a976c6639914af7c7a6
=> HEX: c7aec5d0d81ba8748acac6931e5add6c24b635181443d0b9d2f8aad90d5fc7774c1e7ee451e755831cd02bfaac3204aed8a43dfec8a22cde4db4463db2c35742062a415441f526daecb59bf6af1ecb8cc9827a259401e850e5e07fdc3c1137f16837abc6655c12c454abe0ca85a596e98473172829581235dd95380b06bf6dd06b89118b0003ea044700a5f2c4c106c3

Decrypt it and we will get the flag:

**CSCV2025{DnS_Exf1ltr4ti0nnnnnnnnnnNN!!}**

Thank you for reading my write up!!




