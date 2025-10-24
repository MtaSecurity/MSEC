---
title: CSCV2025 FOR CASEALPHAS

---

FORENSICS CSCV 2025
from: MTA.SUPPORT

FOR3: CASE ALPHAS

First, we have the file zip from admin, then we enter the password to extract this

After extracted, we got 2 files

Open ReadMe.pdf to read the information and get the password to extract file zip

![image](https://hackmd.io/_uploads/HJo28szCxg.png)

Open evidences, there is 3 file, we just need to care about evidences.ad1 first

Open it in FTK Imager: 
![image](https://hackmd.io/_uploads/H1eTwsGRgx.png)

Fortunately, after a quick scan, notice that the insider threat has downloaded chatgpt desktop and simplenote, so that we need to check file .db in 2 files

CHATGPT:

Users/windows/Appdata/Local/OpenAI.../LocalCache/Roaming/IndexedDB/...db

![image](https://hackmd.io/_uploads/B17VosGAel.png)

Then we read the file .log and get the details of the conversation between Hacker and ChatGPT

Amazing, we actually get the recovery key to unlock Bitlocker 

![image](https://hackmd.io/_uploads/SJj0osz0el.png)

Remember the hard disk we have in evidences file, unlock it and we see the file secret.zip

![image](https://hackmd.io/_uploads/Byt6hjMAgg.png)

We still have doubts about Simplenote, so that we need to find information about it 

Simplenote:

Users/windows/Appdata/Local/Automatic…/LocalCache/Roaming/IndexedDB/...leveldb

Then we begin to read the file .log to get information like what we did with Chatgpt

We are lucky again, this file contain a password, i think that this is the password for secret.zip

![image](https://hackmd.io/_uploads/HkZd0ofRgl.png)

It is really working, we have the secret file like this

![image](https://hackmd.io/_uploads/r1TaCiGRlg.png)

Read file ssh.txt, we have an URL and password, open it in browser and get the flag

![image](https://hackmd.io/_uploads/SkVuy3fCgl.png)

FINAL FLAG IS: 

![image](https://hackmd.io/_uploads/rk-GxnzCxe.png)


Thank you for reading my write up <33

