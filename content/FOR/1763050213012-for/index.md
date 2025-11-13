---
title: 'BUCKEYE2025 FORENSICS & MISC: Big Data Analysis & Minecraft'

---

**BUCKEYECTF FORENSICS & MISC WRITEUP**

FOR: Big Data Analysis
MISC: Minecraft

I, Big Data Analysis

![image](https://hackmd.io/_uploads/rJjSjNfeZx.png)

- Đề bài của challenge rất ngắn gọn, tìm tất cả các repository names trong Github create events 2023

- Để có thể giải được challenge này, ta phải truy cập vào một trang có thống kê về số lượng các thể loại của Github của ngày/tháng/năm

- Trang mà tôi muốn nhắc đến đó chính là **Console Google Cloud**
- Ở trong Google Cloud có một chức năng tên là BigQuery giúp chúng ta có thể thống kê được số lượng repository names đề bài mong muốn

![image](https://hackmd.io/_uploads/rJTNhNMlbg.png)

- Sau khi vào phần BigQuery, chúng ta ấn vào tab Request (như trong hình)

![image](https://hackmd.io/_uploads/H1uWa4fe-e.png)

- Sau đó paste lệnh sau để thống kê số repo name được tạo bằng CreateEvent trong Github năm 2023 rồi nhấn Execute

**```SELECT COUNT(DISTINCT repo.name) as unique_repos FROM githubarchive.year.2023 WHERE type = 'CreateEvent'```**

![image](https://hackmd.io/_uploads/HJPSTEzx-x.png)

Như vậy có tổng cộng **63421480** repository names


**Flag chính là:

bctf{63421480}**

II, Minecraft 

![image](https://hackmd.io/_uploads/BJnRA4GeWg.png)

- Đề cho ta một file **minecraft.zip**, và một gợi ý là ta phải dùng **redstone** để giải được bài này, thật tuyệt vời cho một người chơi minecraft 10 năm như tôi hihihihihi

- Đầu tiên chúng ta tải file Minecraft.zip về rồi load file vào Game, tức là Download map mà đề bài cho vào trong game Minecraft của mình (lưu ý dùng bản update mới nhất)

![image](https://hackmd.io/_uploads/BkvaeBGl-e.png)

![image](https://hackmd.io/_uploads/S1VJWHzlZl.png)
Bước vào map, ta thấy vô vàn những đá đỏ chằng chịt nối với nhau, sau khi khám phá một lúc tôi đã thấy vấn đề của challenge này là phải làm sáng được chiếc đèn (sau đó sẽ ra cái gì đó), tôi đào xuống dưới thì không có block command nào cả vậy nên chúng ta bắt buộc phải làm sáng đèn từ những cục đá đỏ kia
![image](https://hackmd.io/_uploads/Sycx-rzgZe.png)

![image](https://hackmd.io/_uploads/HyhNWBfxWg.png)

- Nhận thấy chúng ta phải làm cho cái đuốc kia sáng lên để dẫn điện cho đá đỏ, và để làm cho cái đuốc đen đó sáng lên thì ta phải làm cho 3 ngọn đuốc dưới sáng lên, cứ nối dẫn như vậy cho cả map 
![image](https://hackmd.io/_uploads/Hki8Zrzx-g.png)

- Cách duy nhất để tác động lên số đá đỏ này là nhờ vô vàn những cần gạt kia, vậy nên ta phải gạt cần gạt thật khéo léo sao cho tất cả các đuốc đều có thể tắt

![image](https://hackmd.io/_uploads/HkeKXHzlWe.png)

![image](https://hackmd.io/_uploads/Hy9c-HMeWl.png)

- Sau khi làm tắt hết đuốc, tôi đã làm cho bóng đèn sáng lên, nhưng vẫn chưa có hiện tượng gì xảy ra
- Đột nhiên tôi nghĩ rằng những cần gạt kia đang trong trạng thái **Bật/Tắt** cũng như định nghĩa của **0 với 1**
Vậy nên tôi đã xem như **Gạt cần = 1, Không gạt = 0** và đổi chúng thành dãy nhị phân 8 byte


![image](https://hackmd.io/_uploads/HkdQmrMxbx.png)

- Sau khi có được dãy nhị phân, tôi dùng công cụ cyberchef để đổi sang plaintext và đã ra được flag của challenge

**01100010 01100011 01110100 01100110 01111011 00111000 00110000 00110000 00110001 00110011 00110100 01001110 00110001 00110000 00110110 00110001 01000011 01111101**	

![image](https://hackmd.io/_uploads/Bylf7BGx-x.png)

**Flag chính là: bctf{800134N1061C}**