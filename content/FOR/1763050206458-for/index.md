---
title: 'BUCKEYE2025 FORENSICS: zip2john2zip'

---

**BUCKEYECTF2025 FORENSICS WRITEUP
**
FOR1: Zip2john2zip


![image](https://hackmd.io/_uploads/Sy6R_7fx-l.png)
- Đầu tiên, từ đề bài, ta có thể thấy được vấn đề là người dùng đã quên mật khẩu của 1 file zip nào đó của họ, thứ còn lại của họ chỉ là 1 cái hash và chúng ta phải lần theo dấu vết từ đây

**1, CRACK PASSWORD**
![image](https://hackmd.io/_uploads/HkDMKmMgbg.png)

- Đọc qua hash, ta thấy nó có dạng pkzip vậy nên ta dự đoán file chứa hash này là một file zip bị khóa, suy ra ta cần crack (trong đề gợi ý) để tìm ra pass của file zip

- Không chờ đợi, ta bắt đầu dùng mọi cách để crack file zip (qua hash) bằng các công cụ khác nhau: rockyou.txt, hashcat, john.. 
- ![image](https://hackmd.io/_uploads/Bkf8NVMlZe.png)
`john --show --format=pkzip ~/Desktop/hash.txt`
- Ở đây tôi dùng lệnh john để crack và may mắn đã thành công có được mật khẩu 
**Mật khẩu của file zip chính là “factfinder”**

**2, RECOVER ZIP FILE **

- Vấn đề chính của challenge này không phải crack pass, mà là ta phải recover lại file zip chứa hash đề cho và dùng mật khẩu ta crack được để mở 

- Sau nhiều lần thử không thành công, tôi đã dùng chương trình Python dưới đây để phân tích chính xác định dạng hash cụ thể được cung cấp trong thử thách này. Tập lệnh này xác định chính xác CRC32, kích thước không nén và dữ liệu chính từ hàm băm, sau đó xây dựng cấu trúc tệp ZIP hợp lệ (tạo một file Zip shopee so với hàng thật) mà giải nén có thể hiểu được.

- Dưới đây là chương trình zip.py tôi dùng để recover file zip fake

```
import sys, os, struct, binascii

def extract_core_from_line(line):
    """
    Tìm phần core giữa '$pkzip2$' và '*$/pkzip2$' (nếu có).
    Nếu không có trailing terminator, lấy phần sau '$pkzip2$' tới cuối.
    """
    if "$pkzip2$" not in line:
        raise ValueError("No $pkzip2$ marker found in hash line")
    part = line.split("$pkzip2$", 1)[1]
    if "*$/pkzip2$" in part:
        part = part.split("*$/pkzip2$", 1)[0]
    return part

def create_the_one_true_zip(hash_file, output_zip):
    print("[+] Starting final attempt with improved ZIP builder.")
    try:
        with open(hash_file, "r") as f:
            full_hash_line = f.read().strip()
    except FileNotFoundError:
        print(f"[-] Error: Hash file '{hash_file}' not found.")
        return

    try:
        core = extract_core_from_line(full_hash_line)
        fields = core.split('*')
        # Heuristic indices based on typical zip2john -> adapt if different
        # We expect fields layout something like: <ver>*<method>*...*<uncomp_size>*...*<crc>*...*<hexdata>
        # But because formats vary, we try to be defensive:
        if len(fields) < 8:
            raise ValueError("Parsed pkzip2 core has unexpectedly few fields.")

        # filename: take left part before ':' if exists in the original full line (common format "path:...:$pkzip2$...")
        if ':' in full_hash_line:
            filename_part = full_hash_line.split(':', 1)[0]
            filename = os.path.basename(filename_part).encode('utf-8')
        else:
            # fallback filename
            filename = b"recovered.bin"

        # parse uncompressed size and CRC: in many examples field index 4 -> hex (e.g. '34' -> 0x34 == 52)
        # If parse fails we fallback to len(data_blob) - 12 later.
        try:
            uncompressed_size = int(fields[4], 16)
        except Exception:
            uncompressed_size = None

        # CRC hex usually near fields[6] in many pkzip2 hash outputs — try robustly
        crc_hex = None
        if len(fields) > 6 and all(c in "0123456789abcdefABCDEF" for c in fields[6]) and len(fields[6]) % 2 == 0:
            crc_hex = fields[6]
        else:
            # find first field that looks like 8-hex CRC
            for f in fields:
                if len(f) == 8 and all(c in "0123456789abcdefABCDEF" for c in f):
                    crc_hex = f
                    break

        if crc_hex is None:
            raise ValueError("Could not locate CRC hex in parsed fields.")

        crc = int(crc_hex, 16)  # integer value of CRC

        # data blob likely the last field (long hex string)
        data_blob_hex = fields[-1]
        # ensure even-length hex; trim whitespace/newlines
        data_blob_hex = data_blob_hex.strip()
        if len(data_blob_hex) % 2 != 0:
            # try to drop a trailing character if any
            data_blob_hex = data_blob_hex[:-1]

        try:
            data_blob = binascii.unhexlify(data_blob_hex)
        except Exception as e:
            raise ValueError(f"Failed to parse data blob hex: {e}")

        compressed_size = len(data_blob)

        # if uncompressed_size unknown, guess for "store" method: often compressed_size - 12 (encryption header)
        if uncompressed_size is None:
            guessed = compressed_size - 12 if compressed_size > 12 else compressed_size
            print(f"[!] uncompressed_size not parsed from hash; guessing {guessed}")
            uncompressed_size = guessed

        print("[+] Parsed values:")
        print(f"    filename: {filename!r}")
        print(f"    crc (int): 0x{crc:08x}")
        print(f"    uncompressed_size: {uncompressed_size}")
        print(f"    compressed_size: {compressed_size}")

    except Exception as e:
        print(f"[-] Error parsing hash: {e}")
        return

    # --- Build ZIP file properly ---
    try:
        with open(output_zip, "wb") as f:
            # keep track of local header offset (start of archive)
            local_header_offset = f.tell()  # should be 0

            # --- Local file header ---
            # local file header structure (little-endian)
            # signature (4), ver needed (2), gp flag (2), comp method (2),
            # mod time (2), mod date (2), crc32 (4), comp size (4), uncomp size (4),
            # fname len (2), extra len (2), filename
            LFH_SIG = b'\x50\x4b\x03\x04'
            f.write(LFH_SIG)
            f.write(struct.pack('<H', 20))        # version needed to extract 2.0
            f.write(struct.pack('<H', 0x0001))    # general purpose bit flag (bit0 = encrypted)
            f.write(struct.pack('<H', 0))         # compression method = 0 (store)
            f.write(struct.pack('<H', 0))         # last mod time
            f.write(struct.pack('<H', 0))         # last mod date
            f.write(struct.pack('<I', crc))       # CRC-32 (little-endian via pack)
            f.write(struct.pack('<I', compressed_size))   # compressed size
            f.write(struct.pack('<I', uncompressed_size)) # uncompressed size
            f.write(struct.pack('<H', len(filename)))     # file name length
            f.write(struct.pack('<H', 0))                  # extra field length
            f.write(filename)
            # Write encrypted/compressed data block (whatever came from hash)
            f.write(data_blob)

            # After writing LFH + data, central directory starts here
            central_dir_offset = f.tell()

            # --- Central directory file header for the file ---
            # central dir file header structure:
            # signature(4), ver made by(2), ver needed(2), gp bit flag(2), comp method(2),
            # mod time(2), mod date(2), crc32(4), comp size(4), uncomp size(4),
            # fname len(2), extra len(2), file comment len(2), disk start(2),
            # internal attrs(2), external attrs(4), rel offset local header(4), filename
            CDFH_SIG = b'\x50\x4b\x01\x02'
            f.write(CDFH_SIG)
            f.write(struct.pack('<H', 20))  # version made by
            f.write(struct.pack('<H', 20))  # version needed to extract
            f.write(struct.pack('<H', 0x0001))  # gp bit flag (encrypted)
            f.write(struct.pack('<H', 0))   # compression method
            f.write(struct.pack('<H', 0))   # mod time
            f.write(struct.pack('<H', 0))   # mod date
            f.write(struct.pack('<I', crc))
            f.write(struct.pack('<I', compressed_size))
            f.write(struct.pack('<I', uncompressed_size))
            f.write(struct.pack('<H', len(filename)))  # fname len
            f.write(struct.pack('<H', 0))               # extra len
            f.write(struct.pack('<H', 0))               # file comment len
            f.write(struct.pack('<H', 0))               # disk number start
            f.write(struct.pack('<H', 0))               # internal file attrs
            f.write(struct.pack('<I', 0))               # external file attrs
            f.write(struct.pack('<I', local_header_offset))  # relative offset of local header
            f.write(filename)

            # Now compute central dir size
            end_of_central_dir = f.tell()
            central_dir_size = end_of_central_dir - central_dir_offset

            # --- End of central directory record ---
            EOCD_SIG = b'\x50\x4b\x05\x06'
            f.write(EOCD_SIG)
            f.write(struct.pack('<H', 0))   # number of this disk
            f.write(struct.pack('<H', 0))   # disk where central directory starts
            f.write(struct.pack('<H', 1))   # number of central dir records on this disk
            f.write(struct.pack('<H', 1))   # total number of central dir records
            f.write(struct.pack('<I', central_dir_size))   # size of central dir
            f.write(struct.pack('<I', central_dir_offset)) # offset of start of central dir
            f.write(struct.pack('<H', 0))   # ZIP file comment length

        print(f"[+] Success: wrote '{output_zip}'. Try unziping with password 'factfinder' (if encrypted).")
        # optional sanity print
        print(f"    central_dir_offset={central_dir_offset} central_dir_size={central_dir_size}")
    except Exception as e:
        print(f"[-] Error while writing ZIP: {e}")

if __name__ == "__main__":
    if len(sys.argv) != 3:
        print(f"Usage: {sys.argv[0]} <hash_file> <output_zip>")
        sys.exit(1)
    create_the_one_true_zip(sys.argv[1], sys.argv[2])
```

> Giải thích code: https://docs.google.com/document/d/1A-bNr_RzncD3YZEkca2Vp7unoiLXHjf_eemTdFmCyt4/edit?usp=sharing
- Chạy chương trình này với lệnh

`python3 <zip.py> <hash.txt> <restored.zip>`

![image](https://hackmd.io/_uploads/Bk3pYEGebl.png)


- Chúng ta đã thành công tạo ra file restored.zip chứa hash mà đề bài cho, giờ chúng ta dùng mật khẩu **factfinder** đã tìm được ở phần 1 để mở khóa file zip

![image](https://hackmd.io/_uploads/BkB15NGlZx.png)

- Flag nằm trong flag.txt

![image](https://hackmd.io/_uploads/HkzbcVMebe.png)

**Flag chính là: 

bctf{not_all_hashes_are_hashed_equally}**

