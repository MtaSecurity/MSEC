---
title: V1T CTF CHALLENGES

---

V1T CTF CHALLENGES 

MISC: MOOO (maybe Crypto & Forensics)

Đề bài cho ta tải 1 file .jpeg, trong file ảnh là một con bò (cảm giác như sắp bị dắt giống bài TrynaCrack)

![image](https://hackmd.io/_uploads/Syyl_S8kWl.png)

![image](https://hackmd.io/_uploads/rkKfurUybx.png)
![image](https://hackmd.io/_uploads/SkIVdH81Ze.png)

- Sau khi đẩy file ảnh vào kali và sử dụng lệnh exiftool thì ta ra một cái comment dài như đường đến trái tim crush vậy 
- Vì đã từng làm qua một bài Crypto liên quan đến mật mã Moo này nên tôi đã copy toàn bộ phần comment và sử dụng https://www.jdoodle.com/execute-cow-online để decode và được kết quả như bên dưới
![image](https://hackmd.io/_uploads/S1KBFSIk-g.png)

- Sau khi thấy một đoạn có vẻ như là password để steghide cái ảnh mooo.jpeg (vẫn sợ bị lừa), tôi đã thử dùng lệnh sau để tiếp tục tìm thông tin từ ảnh

![image](https://hackmd.io/_uploads/Skg6FrU1Wg.png)

- Và đúng là đã xuất hiện 1 file secret.zip, có vẻ chúng ta đang đi đúng hướng, mở file secret.zip ra sẽ có 2 file là extract.txt và TRYYY.txt

- Mở cả 2 ra đều có dòng chữ TRY HARDER, tôi nghĩ tôi đã bị lừa, nhưng khoan, khi xem kĩ thì file TRYYY.txt chứa tận 10 dòng trong khi đó file extract.txt chỉ có 1 dòng

![image](https://hackmd.io/_uploads/B15q9SUkWg.png)

- Thử sử dụng lệnh nano để sửa file TRYYY.txt, tôi đã phát hiện ra bí mật của file, có vẻ như đây là whitespace encode với các dấu space và tab đan xen nhau
![image](https://hackmd.io/_uploads/HJTZsrLyWg.png)

- Vậy nên tôi đã thử dùng tool stegsnow để decode, cụ thể như bên dưới 
`stegsnow -C TRYYY.txt`
![image](https://hackmd.io/_uploads/B1PjsB8J-g.png)


Như vậy flag của bài MOOO này chính là: **v1t{D0wn_Th3_St3gN0_R4bb1t_H0l3}**

**Cảm ơn vì đã dành thời gian để đọc bài viết của mình <3




