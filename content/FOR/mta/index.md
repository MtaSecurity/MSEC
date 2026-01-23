---
title: FORENSICS

---

# WRITE UP FOR FORENSICS CHALLENGE IN MTA CTF 2026

**Team: vibe_coders**

Xin chào mọi người, cuối tuần vừa rồi, team mình vừa tham gia kỳ thi MTA CTF 2026 và giải quyết được 2 challenges Forensics, và hôm nay mình sẽ chia sẻ cho mọi người cách làm 2 bài này

# 1. Siusiusiu (Anh 7)

![image](https://hackmd.io/_uploads/BJv8V_0Sbl.png)

- Từ đề bài ta có thể xác định được một số thông tin:

    -  Flag được chia nhỏ thành nhiều phần khác nhau
    -  File được ghi lại duy nhất là một file pcap (thường chưa thông tin liên quan đến các log và lịch sử lưu lượng mạng của hacker)

**1. Phân tích file `challenge.pcap`**

- Ngay sau khi mở file trên Wireshark, ta có thể thấy được một dãy lượt truy cập với toàn bộ protocol là **ICMP** 
![image](https://hackmd.io/_uploads/HkveUOCSZl.png)
- Vì toàn bộ protocol là ICMP, ta có thể sớm xác định được dấu hiệu của data exfiltration, do ICMP thường được dùng để che giấu dữ liệu, exfiltrate thông tin và bypass firewall (vì ping thường được cho phép)
- Vì ICMP chỉ được dùng cho ping, báo lỗi mạng, kiểm tra kết nối, vậy nên data khả năng cao sẽ nằm trong raw byte

-> Kết luận về file .pcap, chúng ta phải bóc tách dữ liệu có trong raw byte từ đó sẽ tìm kiếm thêm được thông tin để giải quyết bài forensics này

**2. Bóc tách và xử lí dữ liệu từ Raw byte**

2.1 Tách dữ liệu từ raw file

- Trước tiên chúng ta xác định chỉ lấy các dữ liệu từ gói ping, vì các ICMP khác không chứa các payload liên quan đến bài

```
 tshark -r siuu.pcap \
-Y "icmp.type==8 || icmp.type==0" \
-T fields \
-e frame.number -e data.data \
> icmp_payload.txt
```

- Giải thích:
    
    - Đọc file pcap offline, giữ nguyên thứ tự frame 
    - type 8 = Echo Request, type 0 = Echo Reply
    - Xuất dữ liệu dạng bảng, không in full packet, không in header dư thừa
    - **data.data** = payload raw của packet (giúp data lấy ra clean hơn)

-> Định hướng rõ ràng: dữ liệu bị exfiltrate qua ping

- Sau khi đã có được file `icmp_payload.txt` chứa dữ liệu được trích từ payload bởi các gói ping, ta sẽ sắp xếp lại payload theo thứ tự gửi đồng thời tách riêng payload:: 

```
sort -n icmp_payload.txt > icmp_payload_sorted.txt
cut -f2 icmp_payload_sorted.txt > payload_only.txt
```

> Ta không ghép các payload thành file lớn, vì trong khi mở file .txt, ta phát hiện ra rằng có rất nhiều PNG Header khác nhau, chứng tỏ đây không phải 1 file lớn mà là các mảnh ghép khác nhau (giống như đề bài mô tả)

- Tiếp theo, ta dùng đoạn code Python sa (đã giải thích trong code) để tách các file thành png riêng biệt:

```
python3 << 'EOF'
import re, os

hexdata = open("payload_only.txt", errors="ignore").read().lower() #đọc file hex
hexdata = re.sub(r'[^0-9a-f]', '', hexdata)

# tách theo FILE1, FILE2, FILE3...
parts = re.split(r'46494c453\d', hexdata) #Split theo marker FILEX

os.makedirs("out_png", exist_ok=True) #Chuẩn bị xuất PNG

png_header = "89504e470d0a1a0a"
idx = 1

#Trích từng PNG
for part in parts
    i = part.find(png_header)
    if i == -1:
        continue
    png_hex = part[i:]
    if len(png_hex) % 2:
        png_hex = png_hex[:-1]
    with open(f"out_png/file_{idx}.png", "wb") as f: #Ghi file
        f.write(bytes.fromhex(png_hex))
    print(f"wrote out_png/file_{idx}.png")
    idx += 1
EOF
```
- Sau khi chạy code Python, ta thu được 5 ảnh giống nhau như sau:
![image](https://hackmd.io/_uploads/rkME6ORHZg.png)
- Ta sẽ dùng thử bộ công cụ stegnography để kiểm tra xem có flag không
- Sau khi thử dùng lệnh **zsteg**, ta ra được kết quả như sau
![image](https://hackmd.io/_uploads/HyqBCu0BWe.png)
- Từ các thông tin ta có sau lệnh zteg, ta thấy rằng trong output zsteg của mỗi ảnh ansX.png có một dòng rất sạch và có cấu trúc:

>     ans1: 1|<số rất dài>#####
>     ans2: 2|<số rất dài>#####
>     …

-> Pattern này không giống ciphertext random (vì có format cố định, có “x|y”, có delimiter). Điều này gợi ý cho ta về thuật toán SSS (Từ những mảnh ghép, ta ghép lại để được secret S ban đầu)

- Sử dụng code Python này để thực hiện hóa ý tưởng khôi phục bằng thuật toán SSS, (xi, yi được lấy từ thông tin của ảnh)

```
from fractions import Fraction

xs = [1,2,3,4,5]
ys = [
  4001196763488924671398131275468270947577983092127414699312722109345497321854384757736409761489384773302104224656628565761564044788152149526926246685045373180,
  6656589208342098749449863462651797245099146039404332171776791812919552539502638440432409969425680496717935317085705707461762863481602901498147275508828715701,
  1101379674428912519173295762469185675294053541687447007997746296715475172037791772710981955424863720953889997580973310389610298832722411850864834954859095721,
  1065163482010585410532229773083222672701576199263370026764512479104351586255156858817245000809843555964560888925393090619350326840943968210226981605366627542,
  6547940631087117423526665494493908237321714012132101228077090360086181782154733698751199105580620001749947991118965048150982947506267570576233715460351311164
]

def lagrange_at(x0):
    total = Fraction(0,1)
    for i,(xi,yi) in enumerate(zip(xs,ys)):
        num = Fraction(1,1)
        den = Fraction(1,1)
        for j,xj in enumerate(xs):
            if j==i: 
                continue
            num *= Fraction(x0 - xj, 1)
            den *= Fraction(xi - xj, 1)
        total += Fraction(yi,1) * num/den
    return total

s0 = int(lagrange_at(0))   # secret = f(0) (ra số âm)
# đổi sang 2's complement để ra bytes chứa flag
nbytes = 66
twos = (1<<(8*nbytes)) + s0
b = twos.to_bytes(nbytes, "big")

start = b.find(b"MSEC")
print(b[start:])          # sẽ thấy flag + 1 byte rác cuối
print(b[start:-1].decode())  # bỏ byte rác
```

- Sau khi chạy dòng code, ta sẽ được kết quả như sau
![image](https://hackmd.io/_uploads/Byg6weFRHbx.png)

-> Flag cần tìm là: **MSEC{LSB_Steganography_and_SSS_is_cool!}**

# II, Bài này dễ nhưng new tech 

![image](https://hackmd.io/_uploads/B1UtmF0H-g.png)

- Từ đề bài ta có thể xác định được một số thông tin:

    -  Bài này không cầ
    -  File được ghi lại duy nhất là một file pcap (thường chưa thông tin liên quan đến các log và lịch sử lưu lượng mạng của hacker)

**1. Phân tích file chall.pcapng**

- Mở file chall.pcapng, trước mắt chúng ta là lượng log khổng lồ với đa số protocol là `USB`, với info `URB_INTERRUPT in`

   ![image](https://hackmd.io/_uploads/SJTxVKCSbg.png)

    -  Việc nó nằm ở interface USBPcap3 cho thấy đây là một hub cụ thể trên máy nạn nhân
-  Bản chất của "Interrupt In"
    - Khác với USB Bulk Transfer (dùng cho USB chứa dữ liệu, ổ cứng) hay Control Transfer (dùng để thiết lập thiết bị), Interrupt In là cơ chế dành cho các thiết bị cần sự phản hồi nhanh và liên tục với hướng đối tượng hầu như là Bàn phím (Keyboard) hoặc Chuột (Mouse)

-> Từ thông tin URB_INTERRUPT IN, hướng điều tra sẽ rẽ vào hai con đường:
>  Hướng A: Mouse Plotting (Nếu là 4-5 byte)
>  Hướng B: Keystroke Injection/Logging (Nếu là 8 byte)

**2. Phân tích sự cố xảy ra**

1. Từ dữ liệu có trong file chall.pcapng, ta thấy được:

    1.1 Dữ liệu rác (Noise): Device 33024 gửi liên tục 4096 gói tin. Nếu là bàn phím, tốc độ gõ này là vô lý.
    1.2 Dấu hiệu chuột (Mouse Signature): Device 33280 chỉ gửi 52 gói tin chứa 01 00... (Click) và 00 00... (Release). Đây là hành vi của Mouse Click.
    - **Kết luận:** Device 33024 là tọa độ chuột (Movement), và Device 33280 là trạng thái click. Flag không được gõ, mà được **VẼ**.

    1.3 Một cách khác để chứng minh hacker đã không sử dụng key strogger để ẩn flag, đó là:
    1.4 Những byte như B0 37 và 8C 50. Trong bảng mã HID Keyboard, không có phím nào mang mã 0xB0 hay 0x8C mà lại được gõ liên tục như vậy. 
    -> Nếu cố ép nó vào script Keylogger, sẽ chỉ nhận được một đống ký tự rác .

-> Đây là kỹ thuật **Mouse Plotting**. Vậy nên để đọc được nó, ta phải tái tạo lại đường đi của con trỏ chuột.

2. Nhưng vấn đề khác lại xuất hiện, ta phải xác định được rằng liệu nó thật sự là chuột hay bảng vẽ:
    2.1 Đầu tiên sử dụng lệnh sau để đếm USB device_address: 
  
     ```
     tshark -r chall.pcapng \
      Y "usb.transfer_type==0x01 && (usb.endpoint_address & 0x80)" \
      T fields -e usb.device_address | sort | uniq -c
    ```
    ![image](https://hackmd.io/_uploads/BJTwuv1I-x.png)
    -> Kết quả đầu ra chứng minh rằng tất cả các gói Interrupt IN đều thuộc về cùng **chỉ một USB DEVICE**
  
    2.2 Tiếp tục sử dụng tshark với lệnh sau để kiểm tra đầu ra endpoint:
     ```
     tshark -r chall.pcapng \
     Y "usb.transfer_type==0x01 && (usb.endpoint_address & 0x80)" \
     T fields -e usb.endpoint_address | sort | uniq -c
    ```
    ![image](https://hackmd.io/_uploads/BkhrFwJ8bx.png)
    -> Kết quả nói rằng 0x81 (kênh tọa độ) có cực nhiều gói, trong khi kênh 0x82 (thường là kênh liên quan đến press/button) chỉ có vỏn vẹn 104 gói
    
    2.3 Để tiếp tục xác thực hành vi, ta kiểm tra thử độ dài payload của mỗi đầu ra endpoint:
    ```
    tshark -r chall.pcapng \
    Y "usb.endpoint_address==0x81" \
    T fields -e usb.data_len | sort | uniq -c

    tshark -r chall.pcapng \
    Y "usb.endpoint_address==0x82" \
    T fields -e usb.data_len | sort | uniq -c
    ```

    ![image](https://hackmd.io/_uploads/rkwsqw1U-x.png)

     -> Ở packet `usb.data_len = 8`, có thể thấy rằng 8 byte ở đây không phải keyboard report, mà là custom HID report của Digitizer bởi vì byte 2–7 không phải là HID keycodes hợp lệ
  
    2.4 Nhận định: Khi di chuyển bút, giá trị không nhảy quanh số 0 mà thay đổi tịnh tiến (ví dụ từ 14256 xuống 14220). 
    -> Điều này chứng tỏ thiết bị đang báo cáo vị trí tại tọa độ X, Y cụ thể chứ không phải dịch chuyển
    2.5 Tiếp tục kiểm tra usbhid.data của endpoint 0x82, mỗi dòng = 8 byte HID report.
    ![image](https://hackmd.io/_uploads/BJSU2vJL-l.png)
    - Nhận thấy rằng `01: Pen down, 00: Pen up` -> Chứng minh rằng đây không phải mouse (mouse click luôn đi kèm movement) và cũng không phải keyboard (keyboard không có 01/00 kiểu này)

=> **Kết luận:** Từ 5 dẫn chứng trên có thể khẳng định rằng thiết bị được hacker dùng trong tình huống này là **Digitizer**, vậy nên định hướng tiếp theo của ta là dựng ngược lại code để khôi phục lại bức ảnh

**3. Dựng lại bằng chứng thật**

- Từ những thông tin đã được phân tích, ta có thể dựng lại bức ảnh bằng code Python như sau (giải thích trực tiếp trên dòng code):

```
import struct
import matplotlib.pyplot as plt

def exploit_tablet_data(filename):

    # Đọc toàn bộ file pcapng dưới dạng bytes thô
    with open(filename, 'rb') as f:
        data = f.read()
        
    events = []  # Danh sách event đã trích: timestamp + device + HID report 8 byte
    
    i = 0
    # Duyệt byte-by-byte để "scan" tìm block pcapng phù hợp
    while i < len(data) - 60:

        # Nếu 4 byte tại vị trí i == 06 00 00 00
        # -> tác giả đang giả định đây là "dấu hiệu" của một block nhất định trong pcapng
        # (cách parse này là heuristic, không phải parser pcapng chuẩn)
        if data[i] == 6 and data[i+1] == 0 and data[i+2] == 0 and data[i+3] == 0:

            # p_offset = i+28: nhảy tới vùng mà tác giả kỳ vọng bắt đầu payload USBPcap
            p_offset = i + 28

            # check signature 0x1B00 tại p_offset:
            # -> Đây giống như một "magic bytes" để xác định "đây là record USB" (heuristic)
            if p_offset + 27 < len(data) and data[p_offset] == 0x1B and data[p_offset+1] == 0x00:

                # Lấy timestamp (theo kiểu pcapng: 2 phần high/low 32-bit)
                ts_high = struct.unpack('<I', data[i+12:i+16])[0]
                ts_low  = struct.unpack('<I', data[i+16:i+20])[0]
                timestamp = (ts_high << 32) | ts_low  # timestamp 64-bit
                
                try:
                    # Lấy data_len từ offset cố định: p_offset+23..+26 (little endian uint32)
                    # -> tác giả kỳ vọng đây là length của HID report trong record USB
                    data_len = struct.unpack('<I', data[p_offset+23:p_offset+27])[0]

                    # Chỉ lấy report dài 8 byte (HID report chuẩn của nhiều thiết bị)
                    if data_len == 8:

                        # Lấy device address (2 byte little endian) ở p_offset+20..+21
                        # -> trong capture m dùng: 33024 (tọa độ) và 33280 (click)
                        device_addr = struct.unpack('<H', data[p_offset+20:p_offset+22])[0]

                        # Lấy đúng 8 byte report bắt đầu tại p_offset+27
                        report = data[p_offset+27 : p_offset+27+8]

                        # Lưu event gồm timestamp, device id, report bytes
                        events.append({'ts': timestamp, 'dev': device_addr, 'report': report})
                except:
                    # Nếu offset sai / out-of-range / unpack lỗi thì bỏ qua
                    pass

        # dịch cửa sổ scan tiến 1 byte
        i += 1
        
    # Sắp xếp event theo thời gian để vẽ đúng thứ tự nét bút
    events.sort(key=lambda x: x['ts'])
    
    # Track drawing state: bút đang chạm (True) hay nhấc lên (False)
    drawing = False
    
    # (2 list này thực ra không dùng ở cuối, chỉ là biến dư)
    x_coords = []
    y_coords = []
    
    # strokes = danh sách các nét vẽ (mỗi stroke là list điểm)
    strokes = []
    current_stroke = []  # stroke hiện tại đang ghi
    
    # Lưu tọa độ cuối cùng đã biết để khi pen down thì có điểm bắt đầu
    last_x = 0
    last_y = 0
    
    # Duyệt từng event theo thời gian
    for e in events:
        dev = e['dev']       # device address (33024 hoặc 33280)
        rep = e['report']    # HID report 8 byte
        
        # DEVICE 33280: CLICKS (PEN PRESSURE / TOUCH)
        # Trong bài của m: endpoint/cụm này chỉ ra 01/00 (pen down/up)
        if dev == 33280:
            btn = rep[0]  # byte đầu tiên biểu thị trạng thái

            if btn == 1:  # 01 => Pen Down (chạm bút)
                drawing = True

                # Nếu đã có tọa độ gần nhất thì khởi tạo stroke bằng điểm đó
                # (tránh stroke bắt đầu rỗng khi click đến trước movement)
                if last_x != 0:
                    current_stroke = [(last_x, last_y)]

            else:  # 00 => Pen Up (nhấc bút)
                drawing = False

                # Nếu stroke đủ dài thì lưu lại
                if len(current_stroke) > 1:
                    strokes.append(current_stroke)

                # reset stroke hiện tại
                current_stroke = []
                
        # DEVICE 33024: ABSOLUTE COORDINATES (DIGITIZER)
        # Đây là luồng gửi tọa độ X/Y tuyệt đối
        elif dev == 33024:
            # Parse Little Endian 16-bit
            # Giả định layout report:
            # rep[2:4] = X (low, high)
            # rep[4:6] = Y (low, high)
            # (rep[0], rep[1], rep[6], rep[7] là byte trạng thái/khác)
            x = rep[2] | (rep[3] << 8)
            y = rep[4] | (rep[5] << 8)
            
            # Cập nhật tọa độ "last seen"
            last_x = x
            last_y = y
            
            # Nếu đang pen-down thì append điểm vào stroke hiện tại
            if drawing:
                current_stroke.append((x, y))
    
    # Nếu file kết thúc khi vẫn đang pen-down thì chốt stroke cuối
    if drawing and len(current_stroke) > 1:
        strokes.append(current_stroke)
        
    print(f"[*] EXTRACTED {len(strokes)} PEN STROKES.")
    
    # --- VISUALIZATION ---
    plt.figure(figsize=(12, 6))
    
    # Tô màu theo thứ tự stroke để nhìn được tiến trình thời gian
    # i/len(strokes) => normalize 0..1
    colors = plt.cm.jet([i/len(strokes) for i in range(len(strokes))])
    
    for idx, seg in enumerate(strokes):
        xs, ys = zip(*seg)  # tách list điểm thành list x và list y

        # Vẽ stroke, dùng màu theo idx, linewidth=2
        plt.plot(xs, ys, color=colors[idx], linewidth=2)
        
    plt.title("SPECTER'S TABLET RECONSTRUCTION (Gradient = Time)")
    plt.axis('equal')          # giữ tỉ lệ trục x=y để không méo hình
    plt.gca().invert_yaxis()   # đảo Y vì hệ tọa độ màn hình thường (0,0) ở góc trên trái

    # Lưu ảnh kết quả
    plt.savefig("flag_tablet.png")
    print("[SUCCESS] TABLET IMAGE SAVED: flag_tablet.png")

# chạy hàm với file pcapng
exploit_tablet_data('chall.pcapng')
```

- Chạy trực tiếp code, ta thu được:
![image](https://hackmd.io/_uploads/rkzz1dyI-l.png)

-> Flag cần tìm của bài Forensics này là: **MSEC{F0r_USBID}**