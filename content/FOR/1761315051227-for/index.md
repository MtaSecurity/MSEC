---
title: CSCV2025 FOR COVERTS

---

FORENSICS CSCV2025 WRITEUP
From: MTA.Support

FOR4: CovertS

First, we have the file and the password provided admin, after extract the file, we have file challenge.pcapng

![image](https://hackmd.io/_uploads/HJb7V4SCgx.png)

Open file, because this file contain 1GB of data, so that we need to predict what protocol is suspicious and start to filter around this

![image](https://hackmd.io/_uploads/rJQhV4BRle.png)

Because the hacker exfiltrated sth, to reduce the possibility of being detected, we often have to split files and data to smaller pieces, so we will need a lot of packets. From there, we can eliminate some protocols such as NTP, ARP, HTTP, UDP

So that there is only protocol TCP left

![image](https://hackmd.io/_uploads/HyDCw4BAee.png)

For ease of analysis, we will open Conservation in Statistics 

From the segment 192.168.192.1:3239, from the user's IP but with many different ports, but each port sends to the same address and has only 1 packet. Very suspicious so we will filter `ip.src == 192.168.203.91 && ip.dst == 192.168.192.1 && tcp.dstport == 3239`

![image](https://hackmd.io/_uploads/S1QWF4BAee.png)

Checking through a few packets, we can see that the tcp.checksum of each packet is readable

![image](https://hackmd.io/_uploads/Hy-E9ESCgg.png)

=> maybe the code has been split into small tcp.checksums, so we immediately extract each tcp.checksum and put them together

Use this tshark code, change the path of your files

`"C:\Program Files\Wireshark\tshark.exe" -r "E:\khang\cscv2025\forensics-Covert-7afc4ba9ad51f576437a2c204831153a\challenge.pcapng" -Y "ip.src == 192.168.203.91 && ip.dst == 192.168.192.1 && tcp.dstport == 3239" -T fields -e tcp.checksum > out.txt`

Open file out.txt, we have a file contain a lot of hex strings so that we use Cyberchef to decode it to base64 and from base64 to plaintext

![image](https://hackmd.io/_uploads/HykMjNB0le.png)

![image](https://hackmd.io/_uploads/HyR7s4SRel.png)


Read the plaintext and get the flag of this challenge

![image](https://hackmd.io/_uploads/S1PYiVHAge.png)


The final flag is: 
**CSCV2025{my_chal_got_leaked_before_the_contest_bruh_here_is_your_new_flag_b8891c4e147c452b8cc6642f10400452}**
