---
title: V1T CTF CHALLENGES

---

**V1T CTF CHALLENGES**

From: VN_MSEC 

Đây là 1 thử thách thuộc giải V1T CTF, liên quan đến crack file zip, sau đó sửa 1 ảnh trong file đó và có gợi ý liên quan đến flag 

1. CRACK FILE 

- Sau khi tải file challenge.zip của giải về, ta sẽ có 1 file trong đó chưa file quackquackquack.png nhưng không thể mở được (đương nhiên rồi vì đề bài yêu cầu ta crack nó)

![image](https://hackmd.io/_uploads/ry6354Lk-x.png)

- Copy file vào kali và bắt đầu crack, tôi đã cố gắng dùng mọi cách để crack như HASHCAT, ROCKYOU, JOHNTHERIPPER nhưng đều không thể được (hoặc do quá lâu nhưng mà tôi ngại đợi hihi), sau đó tôi thử dùng BKCRACK và BOOM! 

![image](https://hackmd.io/_uploads/Hy6N3NLkWl.png)

- Như các bạn thấy, không có gì cả chỉ là nó lâu vl

![image](https://hackmd.io/_uploads/H1BnhN81Ze.png)

- Sau khi đá qua osint (vì quá bất lực) để tìm bài công viên chứa con vịt thì tôi đã quay lại và có kết quả !!

- Nhờ có key thì tôi đã chính thức crack được file và lấy được ảnh quackquackquack.png thật sự (dùng lệnh dưới) 

    `bkcrack -C challenge.zip -c quackquackquack.png -k 4672d551 bcb3adcb c76d52c5 -d quackquackquack1.png`

![image](https://hackmd.io/_uploads/r1Z9REUkWx.png)
![image](https://hackmd.io/_uploads/r12oRVL1Zg.png)

- Nội dung ảnh bảo flag là password và sau khi dùng lệnh exiftool thì ta thấy được flag là “D4mn_br0_H0n3y_p07_7yp3_5h1d”, nhét vào v1t{} và submit thôi

- Nhưng KHÔNG, BTC V1T chắc muốn test độ cay của tôi nên đã ra 1 flag bịp và cái này incorrect thật, vậy nên tôi lại phải ngồi tìm cách để tìm mọi thứ từ cái ảnh này (dùng 1 ticket và không có tác dụng gì)

2. CHỈNH SỬA ẢNH

- Sau khi dùng tất cả các tool liên quan để tìm kiếm từ cái ảnh này, tôi không thể tìm ra bất kì thứ gì (exiftool, zsteg, strings, file, binwalk, kiểm tra hexdump, kiểm tra crc,...)
- Vậy nên tôi đã nghĩ đến rằng có lẽ mở rộng tấm ảnh này ra sẽ có thể xem được gì đó 
-Tôi đã dùng code python sau để mở rộng hình ảnh (cụ thể là sửa chiều cao của ảnh xem có gì đặc biệt không) 

```
import struct
import zlib
import sys

def fix_png_height(input_file, output_file, new_height):
    with open(input_file, 'rb') as f:
        data = f.read()
    
    # PNG signature
    if data[:8] != b'\x89PNG\r\n\x1a\n':
        print("Không phải file PNG hợp lệ!")
        return False
    
    # Tìm IHDR chunk
    ihdr_start = 8
    ihdr_length = struct.unpack('>I', data[ihdr_start:ihdr_start+4])[0]
    
    if data[ihdr_start+4:ihdr_start+8] != b'IHDR':
        print("Không tìm thấy chunk IHDR!")
        return False
    
    # Lấy chiều rộng hiện tại
    current_width = struct.unpack('>I', data[ihdr_start+8:ihdr_start+12])[0]
    current_height = struct.unpack('>I', data[ihdr_start+12:ihdr_start+16])[0]
    
    print(f"Kích thước hiện tại: {current_width} x {current_height}")
    print(f"Đang đổi chiều cao thành: {new_height}")
    
    # Tạo dữ liệu IHDR mới
    ihdr_data = data[ihdr_start+8:ihdr_start+12] + struct.pack('>I', new_height) + data[ihdr_start+16:ihdr_start+8>
    
    # Tính CRC mới
    new_crc = zlib.crc32(b'IHDR' + ihdr_data) & 0xffffffff
    new_crc_bytes = struct.pack('>I', new_crc)
    
    # Xây dựng file mới
    new_data = (data[:ihdr_start+12] + 
                struct.pack('>I', new_height) + 
                data[ihdr_start+16:ihdr_start+8+ihdr_length] + 
                new_crc_bytes + 
                data[ihdr_start+12+ihdr_length:])
```

- Chạy lệnh này để sửa độ cao ảnh thành 1500
`python3 fix_png.py quackquackquack.png 1500`

- File quackquackquack.png sau khi được sửa độ cao thật sự lộ ra một manh mối quan trọng, là một mã morse (và chúng ta đã thành công tôi nghĩ vậy hihi)
![image](https://hackmd.io/_uploads/rJB7ZHLJZl.png)

3. Decode 
- Sau khi lấy được mã morse trong ảnh, tôi đã decode nó trên web và ra được dòng ‘SHA512’

![image](https://hackmd.io/_uploads/SyOO-BUyWl.png)

- Kết luận từ ảnh: “just kidding the real flag is the password in sha512”, vậy nên tôi đã lấy sha512 của dòng flag bịp D4mn_br0_H0n3y_p07_7yp3_5h1d, và ra được kết quả như bên dưới

![image](https://hackmd.io/_uploads/BJSiQHI1Zx.png)

- Vậy nên flag chính xác là 

**v1t{7083748baa3a42dc0a93811e4f5150e7ae1a050a0929f8c304f707c8c44fc95d86c476d11c9e56709edc30eba5f2d82396f426d93870b56b1a9573eaac8d0373}**

Đây là 1 challenge rất hay luyện skills về crack, image và cả decode mà các bạn nên đọc và làm thử tks
