---
title: CCSCV2025 FOR NOSTALGIAS

---

FORENSICS CSCV WRITEUP 2025
From: MTA.Support

FOR2: NostalgiaS

First, after extract the file zip with the password provided by admin, we have file challenge.ad1 and we need to open it in FTK Imager

![image](https://hackmd.io/_uploads/r1OkgoEClg.png)

Remember the user who was hacked name kadoyat, so we go to get information in users/kadoyat

![image](https://hackmd.io/_uploads/BJySeoNClx.png)

Look around for important files like documents, downloads, desktop but there are no any information there so that we need to navigate to find out from his web history

Continuing, we scan all the file in Chrome and Edge but no important information, there is still only Internet Explorer left but nothing too

Next, we go to Webcache to exploit more information from many files .log

![image](https://hackmd.io/_uploads/Hy4W8sV0xe.png)

Fortunately, we find a suspicious line, the users has downloaded FlashInstaller.hta from a github link

https://gist.githubusercontent.com/oumazio/ad5626973af6118062ae401c1e788464/raw/725302cda73d10e260e2ed0f26d935e576d3bc1c/FlashInstaller.hta

Access to this link and look around this web, this is a html program. Besides, there is another strange link from github to a file name something.txt

![image](https://hackmd.io/_uploads/rJAEwsVAgl.png)

Keep continue to investigate from this, we access the link to something.txt and the screen will be like this

![image](https://hackmd.io/_uploads/HkE2PsV0ge.png)

The file something.txt is an obfuscated javascript file, to deobf we can go to obf-io.deobfuscate.io, then we have this

![image](https://hackmd.io/_uploads/rymz_o4Rel.png)

Exploit the Javascript code:

![image](https://hackmd.io/_uploads/S1lLisNRgg.png)



This code means: check if HKCU\\SOFTWARE\\hensh1n exists, if not add a random 8 character value to the hensh1n key

From file NTUSER.dat by user Kadoyat, we find this string is: HxrYJgdu

And in that same code, we can find another link from github and we access to it:
https://gist.githubusercontent.com/oumazio/fdd0b2711ab501b30b53039fa32bc9ca/raw/ca4f9da41c5c64b3b43f4b0416f8ee0d0e400803/secr3t.txt

This script seems like be encrypted by base64 so that we need to decrypt it in Cyberchef

![image](https://hackmd.io/_uploads/SkIox2VCxl.png)

Download hex string from Pastebin then XOR with 0x24. Next, load assembly into RAM ([System.Reflection.Assembly]::Load) to call StealerJanai.core.RiderKick.Run()


To be easy to understand, we use Deepseek to change the code to Python and run in VMWare to be safe (Need to copy the script in Pastebin to flag.txt)

```
import os
import binascii

def analyze_flag_file():
    try:
        # Read flag.txt file
        with open('flag.txt', 'r', encoding='utf-8') as f:
            hex_content = f.read()
        
        print("=== ANALYZING flag.txt FILE ===")
        print(f"Content length: {len(hex_content)} chars")
        
        # Parse hex values (0x12, 0x34, ...)
        hex_values = []
        lines = hex_content.split('\n')
        
        for line_num, line in enumerate(lines, 1):
            parts = line.split(',')
            for part in parts:
                part = part.strip()
                if part.startswith('0x'):
                    hex_values.append(part)
                elif part:  # Non-hex values
                    print(f"Line {line_num}: Non-hex value found: '{part}'")
        
        print(f"Total hex values found: {len(hex_values)}")
        
        if not hex_values:
            print("❌ No hex values found!")
            return None
        
        # Convert hex to bytes
        encoded_bytes = bytearray()
        for i, hex_val in enumerate(hex_values):
            try:
                byte_val = int(hex_val, 16)
                encoded_bytes.append(byte_val)
            except ValueError as e:
                print(f"❌ Error converting hex value {i}: {hex_val} - {e}")
        
        print(f"Encoded bytes length: {len(encoded_bytes)} bytes")
        print(f"First 20 encoded bytes (hex): {encoded_bytes[:20].hex()}")
        
        return encoded_bytes
        
    except FileNotFoundError:
        print("❌ File 'flag.txt' not found!")
        print("Please ensure the file is in the same directory as this script")
        return None
    except Exception as e:
        print(f"❌ Error: {e}")
        return None

def xor_decrypt_and_analyze(encoded_bytes, key=0x24):
    """XOR decrypt and detailed analysis"""
    
    print(f"\n=== XOR DECRYPTION (Key: 0x{key:02x}) ===")
    
    # XOR decrypt
    decrypted_bytes = bytearray()
    for byte_val in encoded_bytes:
        decrypted_bytes.append(byte_val ^ key)
    
    print(f"Decrypted data length: {len(decrypted_bytes)} bytes")
    print(f"First 50 bytes (hex): {decrypted_bytes[:50].hex()}")
    
    # Check PE signature
    if decrypted_bytes[:2] == b'MZ':
        print("✅ VALID PE FILE DETECTED (MZ Header)")
        
        # Find PE header
        pe_offset = decrypted_bytes.find(b'PE\x00\x00')
        if pe_offset != -1:
            print(f"✅ PE header found at offset: 0x{pe_offset:x}")
            
            # Extract basic information from PE header
            try:
                # Machine type
                machine_type = int.from_bytes(decrypted_bytes[pe_offset+4:pe_offset+6], 'little')
                machine_types = {
                    0x014c: "i386",
                    0x8664: "x86-64", 
                    0x01c0: "ARM",
                    0xaa64: "ARM64"
                }
                print(f"Machine Architecture: {machine_types.get(machine_type, f'Unknown: 0x{machine_type:04x}')}")
                
                # Number of sections
                num_sections = int.from_bytes(decrypted_bytes[pe_offset+6:pe_offset+8], 'little')
                print(f"Number of sections: {num_sections}")
                
            except Exception as e:
                print(f"PE header parsing error: {e}")
    else:
        print("❌ NOT a valid PE file")
        # Show raw text for debugging
        try:
            text_preview = decrypted_bytes[:100].decode('utf-8', errors='ignore')
            print(f"Text preview: {text_preview}")
        except:
            pass
    
    return bytes(decrypted_bytes)

def extract_interesting_strings(decrypted_data):
    """Extract interesting strings from binary"""
    
    print(f"\n=== STRING EXTRACTION ===")
    
    strings = []
    current_string = ""
    
    for byte_val in decrypted_data:
        if 32 <= byte_val <= 126:  # Printable ASCII
            current_string += chr(byte_val)
        else:
            if len(current_string) >= 4:  # Only take long strings
                strings.append(current_string)
            current_string = ""
    
    # Filter interesting strings
    keywords = ['http', 'steal', 'pass', 'token', 'key', 'login', 'bank', 'crypto', 
                'wallet', 'metamask', 'discord', 'telegram', 'email', 'user', 'pass',
                'password', 'credit', 'card', 'banking', 'secret']
    
    interesting_strings = []
    for s in strings:
        if any(keyword in s.lower() for keyword in keywords):
            interesting_strings.append(s)
    
    print(f"Found {len(interesting_strings)} interesting strings:")
    for i, s in enumerate(interesting_strings[:30]):  # Show first 30 strings
        print(f"  {i+1:2d}. {s}")
    
    return interesting_strings

def save_decrypted_dll(decrypted_data):
    """Save decrypted DLL for further analysis"""
    filename = "StealerJanai_decrypted.dll"
    
    try:
        with open(filename, 'wb') as f:
            f.write(decrypted_data)
        
        print(f"\n=== FILE SAVED ===")
        print(f"✅ Decrypted DLL saved as: {filename}")
        print(f"📁 File size: {len(decrypted_data)} bytes")
        
        # Verify file
        if os.path.exists(filename):
            file_size = os.path.getsize(filename)
            print(f"📊 Verified file size: {file_size} bytes")
            
    except Exception as e:
        print(f"❌ Error saving file: {e}")

def calculate_hashes(decrypted_data):
    """Calculate file hashes for identification"""
    import hashlib
    
    print(f"\n=== FILE HASHES ===")
    
    # MD5
    md5_hash = hashlib.md5(decrypted_data).hexdigest()
    print(f"MD5:    {md5_hash}")
    
    # SHA-1
    sha1_hash = hashlib.sha1(decrypted_data).hexdigest()
    print(f"SHA-1:  {sha1_hash}")
    
    # SHA-256
    sha256_hash = hashlib.sha256(decrypted_data).hexdigest()
    print(f"SHA-256: {sha256_hash}")

# EXECUTE ANALYSIS
print("🚀 STARTING MALWARE ANALYSIS...")
print("=" * 50)

# 1. Analyze flag.txt file
encoded_data = analyze_flag_file()

if encoded_data:
    # 2. XOR decrypt
    decrypted_dll = xor_decrypt_and_analyze(encoded_data)
    
    # 3. Calculate hashes
    calculate_hashes(decrypted_dll)
    
    # 4. Extract strings
    interesting_strings = extract_interesting_strings(decrypted_dll)
    
    # 5. Save file for further analysis
    save_decrypted_dll(decrypted_dll)
    
    print("\n" + "=" * 50)
    print("🎯 ANALYSIS COMPLETED!")
    print("\n📋 RECOMMENDED NEXT STEPS:")
    print("1. Analyze StealerJanai_decrypted.dll with dnSpy")
    print("2. Scan file on VirusTotal using the hashes above")
    print("3. Analyze behavior in sandbox environment")
    print("4. Check for network indicators in the extracted strings")
else:
    print("❌ Cannot proceed with analysis due to file reading errors!")
```


After get file StealerJanai.dll, load it into DnSpy to analyze this malware code

Look around all the file but nothing surprise me instead of 1 file name “SystemSecretInformation”, open it and analyze the code

1. The DecodeMagicToString() function is a Base62 decode function with the alphabet 0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz 

![image](https://hackmd.io/_uploads/BJHI_3VRex.png)

2. The Collect() function will build a string text + machinename + text2 + registryvalue + }

![image](https://hackmd.io/_uploads/HyT7Y2VRgl.png)



3. RegistryValue is HxrYJgdu, we’ve found it before

4.We can find Machine name in Windows/System32/config then load into RegistryExplorer, the computer name will be in the key HKLM\SYSTEM\CurrentControlSet\Control\ComputerName\ComputerName: DESKTOP-47ICHL6

So that the machine name is: DESKTOP-47ICHL6

Summary, we have:
 
text: CSCV2025{your_computer_
machine name: DESKTOP-47ICHL6
text2: has_be3n_kicked_by
registryvalue: HxrYJgdu 

Finally, the flag is: 

**CSCV2025{your_computer_DESKTOP-47ICHL6_has_be3n_kicked_byHxrYJgdu}**








