---
title: 'Write-up for Reverse Master CSCV2025 '

---

Hello everyone🫡! Today I will write a little bit about The ***Reverse Master*** challenge in CSCV2025.
The question:
![image](https://hackmd.io/_uploads/B1vSBTNAee.png)

When we install the file we will have an apk file like this:
![image](https://hackmd.io/_uploads/Bk60raNReg.png)
Use **jadx-gui** to decompile this file,then open **Source code/com/ctf.challenge** and you will see **Main Activity** like this:
![image](https://hackmd.io/_uploads/BJ7-D6NRlx.png)
Open that and you will see the code below:
```
package com.ctf.challenge;

import android.graphics.Color;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.view.ViewGroup;
import android.widget.Button;
import android.widget.LinearLayout;
import android.widget.TextView;
import androidx.appcompat.app.AppCompatActivity;
import com.google.android.material.textfield.TextInputEditText;
import com.google.android.material.textfield.TextInputLayout;
import java.io.BufferedReader;
import java.io.File;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.InetSocketAddress;
import java.net.Socket;
import o.CountDownTimerC0176z3;
import o.F2;
import o.F3;
import o.RunnableC0154w;
import o.V4;

/* loaded from: classes.dex */
public final class MainActivity extends AppCompatActivity {
    public static final /* synthetic */ int b = 0;
    public final byte[] a = {66, 51, 122, 33, 86};

    static {
        try {
            System.loadLibrary("native-lib");
        } catch (UnsatisfiedLinkError e) {
            Log.e("CTF", "❌ Native lib failed: " + e.getMessage());
        }
    }

    public final native boolean checkSecondHalf(String str);

    public final native String getHint();

    public final void h(LinearLayout linearLayout, String str, String str2) {
        View viewInflate = getLayoutInflater().inflate(android.R.layout.simple_list_item_2, (ViewGroup) linearLayout, false);
        TextView textView = (TextView) viewInflate.findViewById(android.R.id.text1);
        TextView textView2 = (TextView) viewInflate.findViewById(android.R.id.text2);
        textView.setText("⚠️ ".concat(str));
        textView.setTextColor(Color.parseColor("#FF5252"));
        textView.setTextSize(16.0f);
        textView.setTypeface(null, 1);
        textView2.setText(str2);
        textView2.setTextColor(Color.parseColor("#BDBDBD"));
        textView2.setTextSize(13.0f);
        linearLayout.addView(viewInflate);
    }

    @Override // androidx.fragment.app.FragmentActivity, androidx.activity.ComponentActivity, androidx.core.app.ComponentActivity, android.app.Activity
    public final void onCreate(Bundle bundle) throws IOException {
        boolean z;
        String line;
        super.onCreate(bundle);
        setContentView(R.layout.activity_main);
        boolean z2 = false;
        boolean z3 = (getApplicationInfo().flags & 2) != 0;
        String[] strArr = {"/system/app/Superuser.apk", "/sbin/su", "/system/bin/su", "/system/xbin/su", "/data/local/xbin/su", "/data/local/bin/su", "/system/sd/xbin/su", "/system/bin/failsafe/su", "/data/local/su"};
        int i = 0;
        while (true) {
            if (i >= 9) {
                try {
                    Runtime.getRuntime().exec("su");
                    break;
                } catch (Exception unused) {
                    z = false;
                }
            } else if (new File(strArr[i]).exists()) {
                break;
            } else {
                i++;
            }
        }
        z = true;
        try {
            BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(Runtime.getRuntime().exec("ps").getInputStream()));
            do {
                line = bufferedReader.readLine();
                if (line == null) {
                    break;
                } else if (!V4.u(line, "frida") && !V4.u(line, "gum-js-loop")) {
                }
            } while (!V4.u(line, "gmain"));
            z2 = true;
            bufferedReader.close();
        } catch (Exception unused2) {
        }
        int i2 = 27042;
        while (true) {
            if (i2 >= 27053) {
                break;
            }
            try {
                Socket socket = new Socket();
                socket.connect(new InetSocketAddress("127.0.0.1", i2), 100);
                socket.close();
                z2 = true;
                break;
            } catch (Exception unused3) {
                i2++;
            }
        }
        if (!z3 && !z && !z2) {
            Thread thread = new Thread(new RunnableC0154w(8, this));
            thread.setDaemon(true);
            thread.start();
            final TextInputLayout textInputLayout = (TextInputLayout) findViewById(R.id.flagInputLayout);
            final TextInputEditText textInputEditText = (TextInputEditText) findViewById(R.id.flagInput);
            Button button = (Button) findViewById(R.id.checkButton);
            Button button2 = (Button) findViewById(R.id.hintButton);
            button.setOnClickListener(new View.OnClickListener() { // from class: o.y3
                /* JADX WARN: Removed duplicated region for block: B:4:0x001a  */
                @Override // android.view.View.OnClickListener
                /*
                    Code decompiled incorrectly, please refer to instructions dump.
                    To view partially-correct code enable 'Show inconsistent code' option in preferences
                */
                public final void onClick(android.view.View r13) {
                    /*
                        r12 = this;
                        r13 = 16
                        r0 = 1
                        int r1 = com.ctf.challenge.MainActivity.b
                        com.google.android.material.textfield.TextInputEditText r1 = r1
                        android.text.Editable r1 = r1.getText()
                        java.lang.String r1 = java.lang.String.valueOf(r1)
                        com.ctf.challenge.MainActivity r2 = r2
                        java.lang.String r3 = "CSCV2025{"
                        boolean r3 = r1.startsWith(r3)
                        r4 = 0
                        if (r3 != 0) goto L1c
                    L1a:
                        r13 = r4
                        goto L6e
                    L1c:
                        java.lang.String r3 = "}"
                        boolean r3 = r1.endsWith(r3)
                        if (r3 != 0) goto L25
                        goto L1a
                    L25:
                        int r3 = r1.length()
                        int r3 = r3 - r0
                        r5 = 9
                        java.lang.String r1 = r1.substring(r5, r3)
                        java.lang.String r3 = "substring(...)"
                        o.F2.e(r1, r3)
                        java.lang.String r5 = r1.substring(r4, r13)
                        o.F2.e(r5, r3)
                        byte[] r6 = new byte[r13]
                        r6 = {x0090: FILL_ARRAY_DATA , data: [122, 86, 27, 22, 53, 35, 80, 77, 24, 98, 122, 7, 72, 21, 98, 114} // fill-array
                        byte[] r7 = new byte[r13]
                        r8 = r4
                    L44:
                        if (r8 >= r13) goto L55
                        r9 = r6[r8]
                        byte[] r10 = r2.a
                        int r11 = r10.length
                        int r11 = r8 % r11
                        r10 = r10[r11]
                        r9 = r9 ^ r10
                        byte r9 = (byte) r9
                        r7[r8] = r9
                        int r8 = r8 + r0
                        goto L44
                    L55:
                        java.lang.String r6 = new java.lang.String
                        java.nio.charset.Charset r8 = o.X.a
                        r6.<init>(r7, r8)
                        boolean r5 = r5.equals(r6)
                        if (r5 != 0) goto L63
                        goto L1a
                    L63:
                        java.lang.String r13 = r1.substring(r13)
                        o.F2.e(r13, r3)
                        boolean r13 = r2.checkSecondHalf(r13)
                    L6e:
                        com.google.android.material.textfield.TextInputLayout r1 = r3
                        if (r13 == 0) goto L80
                        java.lang.String r13 = "🎉 Correct! Flag is valid!"
                        android.widget.Toast r13 = android.widget.Toast.makeText(r2, r13, r0)
                        r13.show()
                        r13 = 0
                        r1.setError(r13)
                        return
                    L80:
                        java.lang.String r13 = "❌ Wrong flag! Try again!"
                        android.widget.Toast r13 = android.widget.Toast.makeText(r2, r13, r4)
                        r13.show()
                        java.lang.String r13 = "Invalid flag"
                        r1.setError(r13)
                        return
                    */
                    throw new UnsupportedOperationException("Method not decompiled: o.ViewOnClickListenerC0170y3.onClick(android.view.View):void");
                }
            });
            button2.setOnClickListener(new F3(3, this));
            return;
        }
        setContentView(R.layout.security_warning);
        TextView textView = (TextView) findViewById(R.id.countdownText);
        LinearLayout linearLayout = (LinearLayout) findViewById(R.id.issuesList);
        if (z3) {
            F2.c(linearLayout);
            h(linearLayout, "Debug Mode Detected", "Application is running in debuggable mode");
        }
        if (z) {
            F2.c(linearLayout);
            h(linearLayout, "Root Access Detected", "Device has been rooted or jailbroken");
        }
        if (z2) {
            F2.c(linearLayout);
            h(linearLayout, "Frida Framework Detected", "Dynamic instrumentation tool is running");
        }
        new CountDownTimerC0176z3(textView, this).start();
    }

    public final native void startFridaMonitoring();
}
```
You can find the first half of the flag with this XOR code from python:
```
byte[] encrypted = {
  122, 86, 27, 22, 53, 35, 80, 77,
  24, 98, 122, 7, 72, 21, 98, 114
};
byte[] key = {66, 51, 122, 33, 86};

for (int i = 0; i < 16; i++) {
    decrypted[i] = encrypted[i] ^ key[i % key.length];
}
String expected = new String(decrypted, Charset.UTF_8);
return input.equals(expected);

```
The output is:
```
8ea7cac794842440

```
To find out the second half of the flag, we'll disassemble and analyze the native code to reverse the logic inside checkSecondHalf from **libnative-lib\.so**
Let use **apktool** command in terminal to extract **libnative-lib\.so** from the apk file:
```
apktool d -f reverse-master.apk -o reverse-master_src

```
![image](https://hackmd.io/_uploads/r1s7bAVAle.png)

A new folder named **reverse-master_src** will appear in the same path you use Terminal. Check that out and find **libnative-lib\.so** in the lib folder
![image](https://hackmd.io/_uploads/B1tjbRERxl.png)
Let open this file in **ghidra** and find the **Java_com_ctf_challenge_MainActivity_checkSecondHalf** function:
```
bool Java_com_ctf_challenge_MainActivity_checkSecondHalf
               (long *param_1,undefined8 param_2,undefined8 param_3)

{
  int iVar1;
  int iVar2;
  int iVar3;
  ulong uVar4;
  char *__s;
  size_t sVar5;
  
  uVar4 = FUN_00119ca0();
  if ((uVar4 & 1) != 0) {
    __android_log_print(4,"Lib-Native","Debugger detected in native code!");
    return false;
  }
  __s = (char *)(**(code **)(*param_1 + 0x548))(param_1,param_3,0);
  if (__s != (char *)0x0) {
    sVar5 = strlen(__s);
    iVar1 = rand();
    iVar1 = iVar1 % 0x32 + 1;
    iVar2 = rand();
    iVar2 = iVar2 % 0x32 + 1;
    if (iVar1 * iVar1 + iVar2 * iVar2 == (iVar2 + iVar1) * (iVar2 + iVar1) + iVar1 * iVar2 * -2 + 1)
    {
      FUN_0011ac60(__s);
      FUN_0011accc();
    }
    iVar1 = rand();
    iVar1 = iVar1 % 100;
    iVar2 = rand();
    iVar2 = iVar2 % 100;
    if ((iVar2 + iVar1) * (iVar2 + iVar1) < iVar1 * iVar1 + iVar2 * iVar2) {
      iVar3 = 0;
    }
    else {
      iVar3 = FUN_0011ad68(__s,sVar5 & 0xffffffff);
      iVar1 = rand();
      iVar1 = iVar1 % 0x32 + 1;
      iVar2 = rand();
      iVar2 = iVar2 % 0x32 + 1;
      if (iVar1 * iVar1 + iVar2 * iVar2 ==
          (iVar2 + iVar1) * (iVar2 + iVar1) + iVar1 * iVar2 * -2 + 1) {
        FUN_0011b5e8(__s);
        FUN_0011b658();
      }
    }
    (**(code **)(*param_1 + 0x550))(param_1,param_3,__s);
    if (iVar3 != 0) {
      iVar1 = rand();
      iVar1 = iVar1 % 100;
      iVar2 = rand();
      iVar2 = iVar2 % 100;
      if (iVar1 * iVar1 + iVar2 * iVar2 <= (iVar2 + iVar1) * (iVar2 + iVar1)) {
        return true;
      }
    }
    iVar1 = rand();
    iVar1 = iVar1 % 0x32 + 1;
    iVar2 = rand();
    iVar2 = iVar2 % 0x32 + 1;
    return iVar1 * iVar1 + iVar2 * iVar2 ==
           (iVar2 + iVar1) * (iVar2 + iVar1) + iVar1 * iVar2 * -2 + 1;
  }
  rand();
  rand();
  return false;
}

```
Let chatGPT analyze this:
![image](https://hackmd.io/_uploads/S1gEWJHCxe.png)
So let go to **FUN_0011ad68** to see what inside this:
```

undefined4 FUN_0011ad68(byte *param_1,int param_2)

{
  byte bVar1;
  byte bVar2;
  byte bVar3;
  byte bVar4;
  byte bVar5;
  undefined1 auVar6 [16];
  undefined8 uVar7;
  uint5 uVar8;
  int iVar9;
  int iVar10;
  byte *pbVar11;
  uint5 *puVar12;
  uint uVar13;
  byte bVar14;
  byte bVar15;
  byte bVar16;
  byte bVar17;
  byte bVar18;
  undefined4 local_90;
  
  pbVar11 = (byte *)calloc(0x10,1);
  uVar13 = 0x1a2b;
  bVar18 = 0;
  local_90 = 0;
  do {
    if (uVar13 < 0x7a8b) {
      if (uVar13 == 0x1a2b) {
        iVar9 = rand();
        iVar9 = iVar9 % 100;
        iVar10 = rand();
        iVar10 = iVar10 % 100;
        uVar13 = 0xbecf;
        if (iVar9 * iVar9 + iVar10 * iVar10 <= (iVar10 + iVar9) * (iVar10 + iVar9)) {
          uVar13 = 0x3c4d;
        }
      }
      else if (uVar13 == 0x3c4d) {
        if (param_2 != 0x10) goto LAB_0011b004;
        iVar9 = rand();
        iVar9 = iVar9 % 0x32 + 1;
        iVar10 = rand();
        iVar10 = iVar10 % 0x32 + 1;
        uVar13 = 0xbecf;
        if (iVar9 * iVar9 + iVar10 * iVar10 !=
            (iVar10 + iVar9) * (iVar10 + iVar9) + iVar9 * iVar10 * -2 + 1) {
          uVar13 = 0x5e6f;
        }
      }
      else {
        if (uVar13 != 0x5e6f) goto LAB_0011b538;
        puVar12 = (uint5 *)calloc(5,1);
        if (puVar12 == (uint5 *)0x0) {
          bVar14 = 0;
          bVar18 = 0;
          bVar16 = 0;
          bVar17 = 0;
          bVar15 = 0;
        }
        else {
          bVar14 = 99;
          bVar15 = 0x7d;
          bVar17 = 0xe2;
          bVar16 = 0x14;
          bVar18 = 0xb8;
          *(undefined4 *)puVar12 = 0xb814e27d;
          *(byte *)((long)puVar12 + 4) = 99;
        }
        bVar1 = *(byte *)((long)puVar12 + 1);
        bVar2 = (byte)*puVar12;
        bVar3 = *(byte *)((long)puVar12 + 2);
        bVar4 = *(byte *)((long)puVar12 + 3);
        bVar5 = *(byte *)((long)puVar12 + 4);
        uVar8 = *puVar12;
        pbVar11[1] = (bVar17 ^ 0x6c) - 10 ^ bVar1;
        *pbVar11 = (bVar15 ^ 0x2f) - 7 ^ bVar2;
        pbVar11[2] = ((bVar16 | 1) ^ 0x95) - 0xd ^ bVar3 ^ 2;
        pbVar11[0xd] = (bVar18 ^ 8) - 0x2e ^ bVar4 ^ 0xd;
        pbVar11[4] = (bVar14 ^ 0x74) - 0x13 ^ bVar5 ^ 4;
        pbVar11[0xf] = (bVar15 ^ 7) - 0x34 ^ bVar2 ^ 0xf;
        pbVar11[0x10] = 0;
        pbVar11[3] = ((bVar18 | 2) ^ 0x21) - 0x10 ^ bVar4;
        pbVar11[0xe] = (bVar14 ^ 0x5a) - 0x31 ^ bVar5 ^ 0xe;
        auVar6._5_3_ = 0;
        auVar6._0_5_ = uVar8;
        auVar6[8] = bVar2;
        auVar6[9] = bVar1;
        auVar6[10] = bVar3;
        auVar6[0xb] = bVar4;
        auVar6[0xc] = bVar5;
        auVar6._13_3_ = 0;
        uVar7 = a64_TBL(ZEXT816(0),auVar6,0x201000403020100);
        bVar18 = ((bVar18 | 7) ^ 0x4d) - 0x1f ^ 8 ^ (byte)((ulong)uVar7 >> 0x18);
        *(ulong *)(pbVar11 + 5) =
             CONCAT17(((bVar16 | 0xb) ^ 0x53) - 0x2b ^ 0xc ^ (byte)((ulong)uVar7 >> 0x38),
                      CONCAT16((bVar17 ^ 0xe2) - 0x28 ^ 0xb ^ (byte)((ulong)uVar7 >> 0x30),
                               CONCAT15((bVar15 ^ 0x17) - 0x25 ^ 10 ^ (byte)((ulong)uVar7 >> 0x28) ,
                                        CONCAT14(((bVar14 | 8) ^ 0x45) - 0x22 ^ 9 ^
                                                 (byte)((ulong)uVar7 >> 0x20),
                                                 CONCAT13(bVar18,CONCAT12((bVar16 ^ 0x28) - 0x1c ^  7
                                                                          ^ (byte)((ulong)uVar7 >>
                                                                                  0x10),
                                                                          CONCAT11(((bVar17 | 5) ^
                                                                                   0x47) - 0x19 ^ 6
                                                                                   ^ (byte)((ulong)
                                                  uVar7 >> 8),
                                                  (bVar15 ^ 0x4c) - 0x16 ^ 5 ^ (byte)uVar7)))))));
        iVar9 = rand();
        iVar9 = iVar9 % 100;
        iVar10 = rand();
        iVar10 = iVar10 % 100;
        uVar13 = 0xbecf;
        if (iVar9 * iVar9 + iVar10 * iVar10 <= (iVar10 + iVar9) * (iVar10 + iVar9)) {
          uVar13 = 0x7a8b;
        }
      }
    }
    else if (uVar13 < 0xbecf) {
      if (uVar13 == 0x7a8b) {
        if (((((((((((((((*pbVar11 == *param_1 && pbVar11[1] == param_1[1]) &&
                        pbVar11[2] == param_1[2]) && pbVar11[3] == param_1[3]) &&
                      pbVar11[4] == param_1[4]) && pbVar11[5] == param_1[5]) &&
                    pbVar11[6] == param_1[6]) && pbVar11[7] == param_1[7]) && bVar18 == param_1[8])
                 && pbVar11[9] == param_1[9]) && pbVar11[10] == param_1[10]) &&
               pbVar11[0xb] == param_1[0xb]) && pbVar11[0xc] == param_1[0xc]) &&
             pbVar11[0xd] == param_1[0xd]) && pbVar11[0xe] == param_1[0xe]) &&
            pbVar11[0xf] == param_1[0xf]) {
          iVar9 = rand();
          iVar9 = iVar9 % 100;
          iVar10 = rand();
          iVar10 = iVar10 % 100;
          if (iVar9 * iVar9 + iVar10 * iVar10 <= (iVar10 + iVar9) * (iVar10 + iVar9)) {
            uVar13 = 0x9cad;
            goto LAB_0011b008;
          }
        }
LAB_0011b004:
        uVar13 = 0xbecf;
      }
      else {
        if (uVar13 != 0x9cad) goto LAB_0011b538;
        local_90 = 1;
LAB_0011b0f4:
        uVar13 = 0xd1e2;
      }
    }
    else {
      if (uVar13 == 0xbecf) {
        local_90 = 0;
        goto LAB_0011b0f4;
      }
      if (uVar13 == 0xd1e2) {
        return local_90;
      }
LAB_0011b538:
      iVar9 = rand();
      iVar9 = iVar9 % 100;
      iVar10 = rand();
      iVar10 = iVar10 % 100;
      if (iVar9 * iVar9 + iVar10 * iVar10 <= (iVar10 + iVar9) * (iVar10 + iVar9)) {
        uVar13 = 0xbecf;
      }
    }
LAB_0011b008:
    rand();
  } while( true );
}
```
So i give this to chatGPT and i have a python code solve the second half like this:

```
# solve_second_half.py
def u8(x): return x & 0xFF

def compute_second_half():
    # puVar12 = 0xb8 14 e2 7d + 0x63 (little-endian 0xb814e27d then 0x63)
    b2 = 0x7d
    b1 = 0xe2
    b3 = 0x14
    b4 = 0xb8
    b5 = 0x63

    # duplicates in code
    b14 = 0x63
    b15 = 0x7d
    b17 = 0xe2
    b16 = 0x14
    b18 = 0xb8

    pb = [0]*16

    pb[1]  = u8(((b17 ^ 0x6C) - 10) ^ b1)
    pb[0]  = u8(((b15 ^ 0x2F) - 7) ^ b2)
    pb[2]  = u8((((b16 | 1) ^ 0x95) - 13) ^ b3 ^ 2)
    pb[13] = u8((((b18 ^ 8) - 46) ^ b4) ^ 13)
    pb[4]  = u8((((b14 ^ 0x74) - 19) ^ b5) ^ 4)
    pb[15] = u8((((b15 ^ 7) - 52) ^ b2) ^ 15)
    pb[3]  = u8((((b18 | 2) ^ 0x21) - 16) ^ b4)
    pb[14] = u8((((b14 ^ 0x5A) - 49) ^ b5) ^ 14)

    # Build auVar6 (first 5 bytes are [b2,b1,b3,b4,b5])
    au = [0]*16
    au[0],au[1],au[2],au[3],au[4] = b2,b1,b3,b4,b5

    # a64_TBL(..., mask=0x0201000403020100) → select bytes [0,1,2,3,4,0,1,2]
    sel = [au[i] for i in (0,1,2,3,4,0,1,2)]
    pick = lambda sh: sel[(sh//8)]

    # update b18 and fill pb[5..12]
    b18 = u8(((b18 | 7) ^ 0x4D) - 0x1F ^ 8 ^ pick(0x18))
    X1 = u8((b15 ^ 0x4C) - 0x16 ^ 5  ^ pick(0x00))
    X2 = u8(((b17 | 5) ^ 0x47) - 0x19 ^ 6  ^ pick(0x08))
    X3 = u8((b16 ^ 0x28) - 0x1C ^ 7  ^ pick(0x10))
    X4 = u8(((b14 | 8) ^ 0x45) - 0x22 ^ 9  ^ pick(0x20))
    X5 = u8((b15 ^ 0x17) - 0x25 ^ 10 ^ pick(0x28))
    X6 = u8((b17 ^ 0xE2) - 0x28 ^ 11 ^ pick(0x30))
    X7 = u8(((b16 | 0xB) ^ 0x53) - 0x2B ^ 12 ^ pick(0x38))

    pb[5],pb[6],pb[7],pb[8],pb[9],pb[10],pb[11],pb[12] = X1,X2,X3,b18,X4,X5,X6,X7

    return bytes(pb)

if __name__ == "__main__":
    second = compute_second_half()
    print(second.hex())       # 6fe3ccc3cf2197e4
    print(second.decode())    # 6fe3ccc3cf2197e4

```
The output is:
```
6fe3ccc3cf2197e4

```
So the flag is CSCV2025{8ea7cac7948424406fe3ccc3cf2197e4}
Thanks for watching this post😘!