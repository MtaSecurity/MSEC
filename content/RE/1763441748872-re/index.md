---
title: Write-up CSCV2025 final REV challenges

---

Hello everyone! After taking a short break, today I will continue by writing about some of the challenges I solved in CSCV2025 competition UwU !

**1. BabyAPK**

**+ Decription:**
![1000002533](https://hackmd.io/_uploads/BkDgcWYlZg.jpg)


**+ Analyst:**
This challenge give us an apk file, Use Jadx-GUI to decompile this file and follow the path Source code\com\svattt.vn\MainActivity to get to the main logic:
```
package com.svattt.vn;

import android.content.Context;
import android.content.res.Resources;
import android.os.Build;
import android.os.Bundle;
import android.util.Base64;
import android.view.View;
import android.view.Window;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;
import androidx.activity.d0;
import androidx.activity.e0;
import androidx.activity.o;
import androidx.activity.p;
import androidx.activity.q;
import androidx.activity.r;
import androidx.activity.s;
import c2.b0;
import com.google.android.material.datepicker.l;
import e.h;
import i0.h0;
import i0.s0;
import java.nio.charset.Charset;
import java.nio.charset.StandardCharsets;
import java.security.InvalidAlgorithmParameterException;
import java.security.InvalidKeyException;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.WeakHashMap;
import javax.crypto.Cipher;
import javax.crypto.NoSuchPaddingException;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import n2.c;

/* compiled from: r8-map-id-808978c3dd8566ee3c602c5e7438c25b04731beab85526738812146b9fed5471 */
/* loaded from: classes.dex */
public class MainActivity extends h {

    /* renamed from: y, reason: collision with root package name */
    public static String f1101y = "";

    /* renamed from: v, reason: collision with root package name */
    public Button f1102v;

    /* renamed from: w, reason: collision with root package name */
    public TextView f1103w;

    /* renamed from: x, reason: collision with root package name */
    public EditText f1104x;

    @Override // e.h, androidx.activity.m, x.f, android.app.Activity
    public final void onCreate(Bundle bundle) throws NoSuchPaddingException, NoSuchAlgorithmException, InvalidKeyException, InvalidAlgorithmParameterException {
        String str;
        Context applicationContext = getApplicationContext();
        int i3 = 2;
        try {
            byte[] bArrN = a0.h.N("736563725f70617274");
            Charset charset = StandardCharsets.UTF_8;
            SecretKeySpec secretKeySpec = new SecretKeySpec(MessageDigest.getInstance("SHA-256").digest((new String(bArrN, charset) + applicationContext.getString(R.string.key_part_res) + "1.0.0").getBytes(charset)), "AES");
            IvParameterSpec ivParameterSpec = new IvParameterSpec(a0.h.N("2557127f6697dca3b9b02c85bc94d1ff"));
            byte[] bArrDecode = Base64.decode("Rh5g13iNfXguQWeoMU9vVc1pvK6oC6/4zLm1aW8nqq/KWQ+GbYACDvBQ9pa75Vy6Rpah4UJunK5WRgwlmE/fcWEPrIKyyh6c/diJrXOGQW8=", 0);
            Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
            cipher.init(2, secretKeySpec, ivParameterSpec);
            str = new String(cipher.doFinal(bArrDecode), charset);
        } catch (Exception e3) {
            e3.printStackTrace();
            str = null;
        }
        f1101y = str;
        super.onCreate(bundle);
        int i4 = o.f100a;
        d0 d0Var = d0.f76a;
        e0 e0Var = new e0(0, 0, d0Var);
        e0 e0Var2 = new e0(o.f100a, o.b, d0Var);
        View decorView = getWindow().getDecorView();
        c.d(decorView, "window.decorView");
        Resources resources = decorView.getResources();
        c.d(resources, "view.resources");
        boolean zBooleanValue = ((Boolean) d0Var.b(resources)).booleanValue();
        Resources resources2 = decorView.getResources();
        c.d(resources2, "view.resources");
        boolean zBooleanValue2 = ((Boolean) d0Var.b(resources2)).booleanValue();
        int i5 = Build.VERSION.SDK_INT;
        s rVar = i5 >= 29 ? new r() : i5 >= 26 ? new q() : new p();
        Window window = getWindow();
        c.d(window, "window");
        rVar.a(e0Var, e0Var2, window, decorView, zBooleanValue, zBooleanValue2);
        setContentView(R.layout.activity_main);
        this.f1102v = (Button) findViewById(R.id.button);
        this.f1104x = (EditText) findViewById(R.id.editTextNumberPassword);
        this.f1103w = (TextView) findViewById(R.id.textView);
        this.f1102v.setOnClickListener(new l(i3, this));
        View viewFindViewById = findViewById(R.id.main);
        b0 b0Var = new b0();
        WeakHashMap weakHashMap = s0.f1654a;
        h0.u(viewFindViewById, b0Var);
    }
}
```
Oh it is AES decrypt with the ciphertext is : "Rh5g13iNfXguQWeoMU9vVc1pvK6oC6/4zLm1aW8nqq/KWQ+GbYACDvBQ9pa75Vy6Rpah4UJunK5WRgwlmE/fcWEPrIKyyh6c/diJrXOGQW8="(base64 we can decrypt after) 

Key have 3 part 
+ Part 1:"736563725f70617274"
ASCII => "secr_part"

+ Part 2:
```
getString(R.string.key_part_res)
```
We need to find key_part_res

+ Part 3:"1.0.0"
=> We still missing a part of the key, after get it we encode to sha-256 to be the final AES key


And IV is: "2557127f6697dca3b9b02c85bc94d1ff".
So we need to find last part of the key, let use text seach to find this:

![image](https://hackmd.io/_uploads/rkhqxJFlWe.png)

Yep, The final part is k3yRe5.
So the complete key is "secr_partk3yRe51.0.0"
Let write a python code to solve this:
```
from hashlib import sha256
from Crypto.Cipher import AES
from base64 import b64decode

key_part = "secr_part" + "k3yRe5" + "1.0.0"
key = sha256(key_part.encode()).digest()

iv = bytes.fromhex("2557127f6697dca3b9b02c85bc94d1ff")
ct = b64decode("Rh5g13iNfXguQWeoMU9vVc1pvK6oC6/4zLm1aW8nqq/KWQ+GbYACDvBQ9pa75Vy6Rpah4UJunK5WRgwlmE/fcWEPrIKyyh6c/diJrXOGQW8=")

cipher = AES.new(key, AES.MODE_CBC, iv)
output = cipher.decrypt(ct)

print(output)
```
Output:
```
b'8f70a0a891efa073a1f430c26ab4ee8d44b44ee25f98a0e095e827b09e5667d6\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10'
```
We get the hash is:
```
8f70a0a891efa073a1f430c26ab4ee8d44b44ee25f98a0e095e827b09e5667d6 
```
but when i submit it is wrong, so i need to find more about the file.

At a moment i think maybe the flag format will appear in the file so i text search "**CSCV2025**" and surprisingly it have a result:
```
package com.google.android.material.datepicker;

import android.util.Log;
import android.view.View;
import android.widget.Toast;
import androidx.appcompat.widget.Toolbar;
import com.svattt.vn.MainActivity;
import j.n3;
import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;

/* compiled from: r8-map-id-808978c3dd8566ee3c602c5e7438c25b04731beab85526738812146b9fed5471 */
/* loaded from: classes.dex */
public final class l implements View.OnClickListener {

    /* renamed from: a */
    public final /* synthetic */ int f952a;
    public final /* synthetic */ Object b;

    public /* synthetic */ l(int i3, Object obj) {
        this.f952a = i3;
        this.b = obj;
    }

    @Override // android.view.View.OnClickListener
    public final void onClick(View view) {
        String string;
        switch (this.f952a) {
            case 0:
                m mVar = (m) this.b;
                int i3 = mVar.V;
                if (i3 == 2) {
                    mVar.B(1);
                    break;
                } else if (i3 == 1) {
                    mVar.B(2);
                    break;
                }
                break;
            case 1:
                e.e eVar = (e.e) this.b;
                eVar.f1225v.obtainMessage(1, eVar.b).sendToTarget();
                break;
            case 2:
                MainActivity mainActivity = (MainActivity) this.b;
                String strTrim = mainActivity.f1104x.getText().toString().trim();
                if (strTrim.isEmpty()) {
                    Toast.makeText(mainActivity, "Please enter passkey !!!", 0).show();
                    break;
                } else if (strTrim.matches("\\d{8}")) {
                    String string2 = null;
                    try {
                        byte[] bArrDigest = MessageDigest.getInstance("SHA-256").digest(strTrim.getBytes(StandardCharsets.UTF_8));
                        StringBuilder sb = new StringBuilder(bArrDigest.length * 2);
                        for (byte b : bArrDigest) {
                            sb.append(String.format("%02x", Integer.valueOf(b & 255)));
                        }
                        string = sb.toString();
                    } catch (Exception e3) {
                        e3.printStackTrace();
                        string = null;
                    }
                    if (string != null && string.equalsIgnoreCase(MainActivity.f1101y)) {
                        try {
                            byte[] bArrDigest2 = MessageDigest.getInstance("SHA-1").digest(strTrim.getBytes(StandardCharsets.UTF_8));
                            StringBuilder sb2 = new StringBuilder(bArrDigest2.length * 2);
                            for (byte b3 : bArrDigest2) {
                                sb2.append(String.format("%02x", Integer.valueOf(b3 & 255)));
                            }
                            string2 = sb2.toString();
                        } catch (Exception e4) {
                            e4.printStackTrace();
                        }
                        if (string2 == null) {
                            string2 = "unknown";
                        }
                        string2 = "CSCV2025{" + string2 + "}";
                    }
                    if (string2 != null) {
                        Toast.makeText(mainActivity, "Unlocked!", 1).show();
                        Log.i("ContentValues", "Derived FLAG: ".concat(string2));
                        mainActivity.f1103w.setText("Logcat ...");
                        break;
                    } else {
                        mainActivity.f1103w.setText("Wrong key.");
                        break;
                    }
                } else {
                    Toast.makeText(mainActivity, "Passkey must be 8 digits (0-9).!", 1).show();
                    break;
                }
                break;
            case 3:
                ((h.a) this.b).a();
                break;
            default:
                n3 n3Var = ((Toolbar) this.b).L;
                i.o oVar = n3Var == null ? null : n3Var.b;
                if (oVar != null) {
                    oVar.collapseActionView();
                    break;
                }
                break;
        }
    }
}
```
Yeah the important part in here:
+ It require 8-digit passkey
+ It convert SHA-256 text we found to 8-digit passkey 
+ This will convert from passkey to SHA-1 (The real flag)

So I wrote a bruce-force code the 8-digit to get the passkey:
```
import hashlib

target = "8f70a0a891efa073a1f430c26ab4ee8d44b44ee25f98a0e095e827b09e5667d6"

for i in range(0,100_000_000):
    s = f"{i:08d}"
    if hashlib.sha256(s.encode()).hexdigest() == target:
        print("FOUND:", s)
        break
```
Output:
```
FOUND: 28109923
```
Use cyberchef to convert into SHA-1
![image](https://hackmd.io/_uploads/ryBB5Jtlbx.png)

So the flag is **CSCV2025{8d1d11a9d8745469e896035df1c94f568abbac32}**

**2) Unlock**

**+Decription:**
![image](https://hackmd.io/_uploads/BJwq51Kgbl.png)

**+Analyst:**

Follow the previous challenge i use Jadx-GUI to decompile this file and follow the path Source code\com\svattt\unlocker\MainActivity to get to the main logic:

```
package com.svattt.unlocker;

import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;
import d1.a;
import f.h;
import f.i;
import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;

/* compiled from: r8-map-id-ab1101ba88990f24aae9a4f9cdbc39df5f16b9691a9e5d423f1390daefbbd936 */
/* loaded from: classes.dex */
public class MainActivity extends i {
    public static String G = "null";
    public Button D;
    public TextView E;
    public EditText F;

    public MainActivity() {
        this.f115i.f1274b.e("androidx:appcompat", new a(this));
        h(new h(this));
    }

    public static String r(String str) {
        try {
            byte[] bArrDigest = MessageDigest.getInstance("SHA-256").digest(str.getBytes(StandardCharsets.UTF_8));
            StringBuilder sb = new StringBuilder(bArrDigest.length * 2);
            for (byte b3 : bArrDigest) {
                sb.append(String.format("%02x", Integer.valueOf(b3 & 255)));
            }
            return sb.toString();
        } catch (Exception e3) {
            e3.printStackTrace();
            return null;
        }
    }

    /* JADX WARN: Removed duplicated region for block: B:62:0x0118  */
    /* JADX WARN: Removed duplicated region for block: B:63:0x011a  */
    /* JADX WARN: Removed duplicated region for block: B:64:0x011d  */
    /* JADX WARN: Removed duplicated region for block: B:71:0x01a8  */
    /* JADX WARN: Removed duplicated region for block: B:73:0x01af  */
    @Override // f.i, androidx.activity.q, z.f, android.app.Activity
    /*
        Code decompiled incorrectly, please refer to instructions dump.
        To view partially-correct code enable 'Show inconsistent code' option in preferences
    */
    public final void onCreate(android.os.Bundle r11) {
        /*
            Method dump skipped, instructions count: 558
            To view this dump change 'Code comments level' option to 'DEBUG'
        */
        throw new UnsupportedOperationException("Method not decompiled: com.svattt.unlocker.MainActivity.onCreate(android.os.Bundle):void");
    }
}
```
Bruh Jadx-GUI warning me that this decompile is not true version, but i see an important function named **r(String str)**:

```
public static String r(String str) {
    try {
        byte[] bArrDigest = MessageDigest.getInstance("SHA-256").digest(str.getBytes(StandardCharsets.UTF_8));
        StringBuilder sb = new StringBuilder(bArrDigest.length * 2);
        for (byte b3 : bArrDigest) {
            sb.append(String.format("%02x", Integer.valueOf(b3 & 255)));
        }
        return sb.toString();
    } catch (Exception e3) {
        e3.printStackTrace();
        return null;
    }
}
```

This function will:
+ It takes SHA-256 from the str
+ Convert to hex lowercase
+ Return back that string

Let follow "r(String)" function to get more information about this code.
Point into "r" next to "(string)" and press X (Find Usage), we get this:
![image](https://hackmd.io/_uploads/S1zeTxKxZe.png)


Click into one of two lines and we get this:

```
package com.google.android.material.datepicker;

import android.util.Log;
import android.view.View;
import android.widget.Toast;
import androidx.appcompat.widget.Toolbar;
import com.svattt.nativelib.NativeLib;
import com.svattt.unlocker.MainActivity;
import com.svattt.unlocker.R;
import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;
import k.x2;

/* compiled from: r8-map-id-ab1101ba88990f24aae9a4f9cdbc39df5f16b9691a9e5d423f1390daefbbd936 */
/* loaded from: classes.dex */
public final class l implements View.OnClickListener {

    /* renamed from: f, reason: collision with root package name */
    public final /* synthetic */ int f1113f;
    public final /* synthetic */ Object g;

    public /* synthetic */ l(int i3, Object obj) {
        this.f1113f = i3;
        this.g = obj;
    }

    @Override // android.view.View.OnClickListener
    public final void onClick(View view) {
        switch (this.f1113f) {
            case 0:
                m mVar = (m) this.g;
                int i3 = mVar.f1116c0;
                if (i3 == 2) {
                    mVar.G(1);
                    mVar.f0.announceForAccessibility(mVar.B().getResources().getString(R.string.mtrl_picker_toggled_to_day_selection));
                    break;
                } else if (i3 == 1) {
                    mVar.G(2);
                    mVar.f1118e0.announceForAccessibility(mVar.B().getResources().getString(R.string.mtrl_picker_toggled_to_year_selection));
                    break;
                }
                break;
            case 1:
                f.e eVar = (f.e) this.g;
                eVar.f1476v.obtainMessage(1, eVar.f1459b).sendToTarget();
                break;
            case 2:
                MainActivity mainActivity = (MainActivity) this.g;
                String strTrim = mainActivity.F.getText().toString().trim();
                if (strTrim.isEmpty()) {
                    Toast.makeText(mainActivity, "Please input passkey !!!", 0).show();
                    break;
                } else if (strTrim.matches("^[A-Za-z0-9]{15}$")) {
                    MainActivity.G = NativeLib.generateInsecureOtp(15);
                    String strR = MainActivity.r(strTrim);
                    String string = null;
                    if (strR != null && strR.equalsIgnoreCase(MainActivity.r(MainActivity.G))) {
                        try {
                            byte[] bArrDigest = MessageDigest.getInstance("SHA-1").digest(strTrim.getBytes(StandardCharsets.UTF_8));
                            StringBuilder sb = new StringBuilder(bArrDigest.length * 2);
                            for (byte b3 : bArrDigest) {
                                sb.append(String.format("%02x", Integer.valueOf(b3 & 255)));
                            }
                            string = sb.toString();
                        } catch (Exception e3) {
                            e3.printStackTrace();
                        }
                        if (string == null) {
                            string = "unknown";
                        }
                        string = "CSCV2025{" + string + "}";
                    }
                    if (string != null) {
                        Toast.makeText(mainActivity, "Unlocked!", 1).show();
                        Log.i("ContentValues", "Derived FLAG: ".concat(string));
                        mainActivity.E.setText("Logcat ...");
                        break;
                    } else {
                        mainActivity.E.setText("Wrong key.");
                        break;
                    }
                } else {
                    Toast.makeText(mainActivity, "Passkey must be 15 characters long, including uppercase, lowercase and numbers.", 1).show();
                    break;
                }
            case 3:
                ((i.a) this.g).a();
                break;
            default:
                x2 x2Var = ((Toolbar) this.g).Q;
                j.o oVar = x2Var == null ? null : x2Var.g;
                if (oVar != null) {
                    oVar.collapseActionView();
                    break;
                }
                break;
        }
    }
}
```



Another passkey checking...
Passkey is created by 15-digit take from generateInsecureOtp (convert from SHA-256 probably).

If passkey is accepted (strTrim == G), it will convert into SHA-1 to the real flag

So we need to find a way to open libnativelib.so in order to locate the hidden data used by **generateInsecureOtp**.
We can use the **tree** command in Linux to list all library files, so we can confirm whether libnativelib.so actually exists.

```
unzip unlocker.apk -d unlocker_unzipped
cd unlocker_unzipped
tree lib
```

We get this:

```
Archive:  unlocker.apk
  inflating: unlocker_unzipped/META-INF/com/android/build/gradle/app-metadata.properties  
  inflating: unlocker_unzipped/META-INF/version-control-info.textproto  
 extracting: unlocker_unzipped/assets/dexopt/baseline.prof  
 extracting: unlocker_unzipped/assets/dexopt/baseline.profm  
  inflating: unlocker_unzipped/classes.dex  
 extracting: unlocker_unzipped/lib/arm64-v8a/libnativelib.so  
 extracting: unlocker_unzipped/lib/armeabi-v7a/libnativelib.so  
 extracting: unlocker_unzipped/lib/x86/libnativelib.so  
 extracting: unlocker_unzipped/lib/x86_64/libnativelib.so  
  inflating: unlocker_unzipped/DebugProbesKt.bin  
  inflating: unlocker_unzipped/META-INF/androidx.activity_activity.version  
  inflating: unlocker_unzipped/META-INF/androidx.annotation_annotation-experimental.version  
  inflating: unlocker_unzipped/META-INF/androidx.appcompat_appcompat-resources.version  
  inflating: unlocker_unzipped/META-INF/androidx.appcompat_appcompat.version  
  inflating: unlocker_unzipped/META-INF/androidx.arch.core_core-runtime.version  
  inflating: unlocker_unzipped/META-INF/androidx.cardview_cardview.version  
  inflating: unlocker_unzipped/META-INF/androidx.constraintlayout_constraintlayout.version  
  inflating: unlocker_unzipped/META-INF/androidx.coordinatorlayout_coordinatorlayout.version  
  inflating: unlocker_unzipped/META-INF/androidx.core_core-ktx.version  
  inflating: unlocker_unzipped/META-INF/androidx.core_core-viewtree.version  
  inflating: unlocker_unzipped/META-INF/androidx.core_core.version  
  inflating: unlocker_unzipped/META-INF/androidx.cursoradapter_cursoradapter.version  
  inflating: unlocker_unzipped/META-INF/androidx.customview_customview.version  
  inflating: unlocker_unzipped/META-INF/androidx.drawerlayout_drawerlayout.version  
  inflating: unlocker_unzipped/META-INF/androidx.dynamicanimation_dynamicanimation.version  
  inflating: unlocker_unzipped/META-INF/androidx.emoji2_emoji2-views-helper.version  
  inflating: unlocker_unzipped/META-INF/androidx.emoji2_emoji2.version  
  inflating: unlocker_unzipped/META-INF/androidx.fragment_fragment.version  
  inflating: unlocker_unzipped/META-INF/androidx.graphics_graphics-shapes.version  
  inflating: unlocker_unzipped/META-INF/androidx.interpolator_interpolator.version  
  inflating: unlocker_unzipped/META-INF/androidx.lifecycle_lifecycle-livedata-core.version  
  inflating: unlocker_unzipped/META-INF/androidx.lifecycle_lifecycle-livedata.version  
  inflating: unlocker_unzipped/META-INF/androidx.lifecycle_lifecycle-process.version  
  inflating: unlocker_unzipped/META-INF/androidx.lifecycle_lifecycle-runtime.version  
  inflating: unlocker_unzipped/META-INF/androidx.lifecycle_lifecycle-viewmodel-savedstate.version  
  inflating: unlocker_unzipped/META-INF/androidx.lifecycle_lifecycle-viewmodel.version  
  inflating: unlocker_unzipped/META-INF/androidx.loader_loader.version  
  inflating: unlocker_unzipped/META-INF/androidx.profileinstaller_profileinstaller.version  
  inflating: unlocker_unzipped/META-INF/androidx.recyclerview_recyclerview.version  
  inflating: unlocker_unzipped/META-INF/androidx.savedstate_savedstate.version  
  inflating: unlocker_unzipped/META-INF/androidx.startup_startup-runtime.version  
  inflating: unlocker_unzipped/META-INF/androidx.tracing_tracing.version  
  inflating: unlocker_unzipped/META-INF/androidx.transition_transition.version  
  inflating: unlocker_unzipped/META-INF/androidx.vectordrawable_vectordrawable-animated.version  
  inflating: unlocker_unzipped/META-INF/androidx.vectordrawable_vectordrawable.version  
  inflating: unlocker_unzipped/META-INF/androidx.versionedparcelable_versionedparcelable.version  
  inflating: unlocker_unzipped/META-INF/androidx.viewpager2_viewpager2.version  
  inflating: unlocker_unzipped/META-INF/androidx.viewpager_viewpager.version  
  inflating: unlocker_unzipped/META-INF/androidx/constraintlayout/constraintlayout-core/LICENSE.txt  
  inflating: unlocker_unzipped/META-INF/com.google.android.material_material.version  
  inflating: unlocker_unzipped/META-INF/kotlinx_coroutines_android.version  
  inflating: unlocker_unzipped/META-INF/kotlinx_coroutines_core.version  
  inflating: unlocker_unzipped/kotlin/annotation/annotation.kotlin_builtins  
  inflating: unlocker_unzipped/kotlin/collections/collections.kotlin_builtins  
  inflating: unlocker_unzipped/kotlin/coroutines/coroutines.kotlin_builtins  
  inflating: unlocker_unzipped/kotlin/internal/internal.kotlin_builtins  
  inflating: unlocker_unzipped/kotlin/kotlin.kotlin_builtins  
  inflating: unlocker_unzipped/kotlin/ranges/ranges.kotlin_builtins  
  inflating: unlocker_unzipped/kotlin/reflect/reflect.kotlin_builtins  
  inflating: unlocker_unzipped/AndroidManifest.xml  
  inflating: unlocker_unzipped/res/-1.xml  
  inflating: unlocker_unzipped/res/-5.xml  
 extracting: unlocker_unzipped/res/-6.webp  
  inflating: unlocker_unzipped/res/-7.xml  
 extracting: unlocker_unzipped/res/-B.png  
  inflating: unlocker_unzipped/res/-B.xml  
 extracting: unlocker_unzipped/res/-N.png  
  inflating: unlocker_unzipped/res/-x.xml  
  inflating: unlocker_unzipped/res/0C.xml  
  inflating: unlocker_unzipped/res/0K.xml  
  inflating: unlocker_unzipped/res/0M.xml  
 extracting: unlocker_unzipped/res/0c.9.png  
  inflating: unlocker_unzipped/res/0w.xml  
 extracting: unlocker_unzipped/res/0x.9.png  
 extracting: unlocker_unzipped/res/1I.9.png  
 extracting: unlocker_unzipped/res/1J.9.png  
  inflating: unlocker_unzipped/res/1N.xml  
  inflating: unlocker_unzipped/res/1R.xml  
 extracting: unlocker_unzipped/res/1e.9.png  
  inflating: unlocker_unzipped/res/21.xml  
  inflating: unlocker_unzipped/res/27.xml  
  inflating: unlocker_unzipped/res/271.xml  
  inflating: unlocker_unzipped/res/2F.xml  
 extracting: unlocker_unzipped/res/2P.png  
  inflating: unlocker_unzipped/res/2R.xml  
 extracting: unlocker_unzipped/res/2d.png  
  inflating: unlocker_unzipped/res/2f.xml  
  inflating: unlocker_unzipped/res/2j.xml  
  inflating: unlocker_unzipped/res/2n.xml  
  inflating: unlocker_unzipped/res/2w.xml  
  inflating: unlocker_unzipped/res/2x.xml  
 extracting: unlocker_unzipped/res/33.9.png  
  inflating: unlocker_unzipped/res/35.xml  
  inflating: unlocker_unzipped/res/3A.xml  
  inflating: unlocker_unzipped/res/3h.xml  
 extracting: unlocker_unzipped/res/3u.9.png  
 extracting: unlocker_unzipped/res/42.9.png  
  inflating: unlocker_unzipped/res/46.xml  
 extracting: unlocker_unzipped/res/49.png  
  inflating: unlocker_unzipped/res/4B.xml  
  inflating: unlocker_unzipped/res/4H.xml  
  inflating: unlocker_unzipped/res/4I.xml  
  inflating: unlocker_unzipped/res/4Q.xml  
  inflating: unlocker_unzipped/res/4S.xml  
  inflating: unlocker_unzipped/res/4_.xml  
  inflating: unlocker_unzipped/res/4j.xml  
 extracting: unlocker_unzipped/res/4k.png  
  inflating: unlocker_unzipped/res/4o.xml  
  inflating: unlocker_unzipped/res/4u.xml  
  inflating: unlocker_unzipped/res/4x.xml  
  inflating: unlocker_unzipped/res/51.xml  
  inflating: unlocker_unzipped/res/59.xml  
 extracting: unlocker_unzipped/res/5D.9.png  
  inflating: unlocker_unzipped/res/5T.xml  
 extracting: unlocker_unzipped/res/5U.png  
  inflating: unlocker_unzipped/res/5l.xml  
  inflating: unlocker_unzipped/res/5z.xml  
  inflating: unlocker_unzipped/res/61.xml  
 extracting: unlocker_unzipped/res/62.9.png  
 extracting: unlocker_unzipped/res/65.9.png  
  inflating: unlocker_unzipped/res/66.xml  
  inflating: unlocker_unzipped/res/68.xml  
  inflating: unlocker_unzipped/res/6Q.xml  
 extracting: unlocker_unzipped/res/6t.png  
  inflating: unlocker_unzipped/res/6x.xml  
 extracting: unlocker_unzipped/res/7C.9.png  
  inflating: unlocker_unzipped/res/7G.xml  
  inflating: unlocker_unzipped/res/7H.xml  
 extracting: unlocker_unzipped/res/7I.9.png  
  inflating: unlocker_unzipped/res/7N.xml  
 extracting: unlocker_unzipped/res/7_.9.png  
 extracting: unlocker_unzipped/res/7i.png  
 extracting: unlocker_unzipped/res/7o.9.png  
  inflating: unlocker_unzipped/res/7s.xml  
  inflating: unlocker_unzipped/res/7s1.xml  
  inflating: unlocker_unzipped/res/80.xml  
  inflating: unlocker_unzipped/res/8a.xml  
 extracting: unlocker_unzipped/res/8h.9.png  
 extracting: unlocker_unzipped/res/8h.png  
  inflating: unlocker_unzipped/res/8s.xml  
  inflating: unlocker_unzipped/res/8y.xml  
 extracting: unlocker_unzipped/res/9N.9.png  
  inflating: unlocker_unzipped/res/9O.xml  
  inflating: unlocker_unzipped/res/9P.xml  
  inflating: unlocker_unzipped/res/9T.xml  
  inflating: unlocker_unzipped/res/9T1.xml  
  inflating: unlocker_unzipped/res/9T2.xml  
  inflating: unlocker_unzipped/res/9V.xml  
 extracting: unlocker_unzipped/res/9X.9.png  
 extracting: unlocker_unzipped/res/9n.9.png  
  inflating: unlocker_unzipped/res/9p.xml  
 extracting: unlocker_unzipped/res/9z.png  
  inflating: unlocker_unzipped/res/9z.xml  
  inflating: unlocker_unzipped/res/A0.xml  
  inflating: unlocker_unzipped/res/A1.xml  
  inflating: unlocker_unzipped/res/A4.xml  
  inflating: unlocker_unzipped/res/A5.xml  
  inflating: unlocker_unzipped/res/Aa.xml  
  inflating: unlocker_unzipped/res/B6.xml  
 extracting: unlocker_unzipped/res/BG.9.png  
  inflating: unlocker_unzipped/res/BJ.xml  
  inflating: unlocker_unzipped/res/BJ1.xml  
 extracting: unlocker_unzipped/res/BL.9.png  
 extracting: unlocker_unzipped/res/BM.png  
  inflating: unlocker_unzipped/res/BT.xml  
  inflating: unlocker_unzipped/res/BT1.xml  
  inflating: unlocker_unzipped/res/BW.xml  
  inflating: unlocker_unzipped/res/Be.xml  
  inflating: unlocker_unzipped/res/By.xml  
 extracting: unlocker_unzipped/res/CK.9.png  
  inflating: unlocker_unzipped/res/Cg.xml  
  inflating: unlocker_unzipped/res/D4.xml  
  inflating: unlocker_unzipped/res/D5.xml  
  inflating: unlocker_unzipped/res/D6.xml  
  inflating: unlocker_unzipped/res/DG.xml  
 extracting: unlocker_unzipped/res/DL.9.png  
  inflating: unlocker_unzipped/res/DS.xml  
  inflating: unlocker_unzipped/res/DZ.xml  
 extracting: unlocker_unzipped/res/D_.9.png  
 extracting: unlocker_unzipped/res/EA.9.png  
 extracting: unlocker_unzipped/res/EP.png  
  inflating: unlocker_unzipped/res/Eg.xml  
  inflating: unlocker_unzipped/res/F8.xml  
  inflating: unlocker_unzipped/res/FS.xml  
 extracting: unlocker_unzipped/res/FW.png  
  inflating: unlocker_unzipped/res/Fd.xml  
  inflating: unlocker_unzipped/res/Fu.xml  
 extracting: unlocker_unzipped/res/G2.9.png  
  inflating: unlocker_unzipped/res/G2.xml  
  inflating: unlocker_unzipped/res/GC.xml  
  inflating: unlocker_unzipped/res/GD.xml  
  inflating: unlocker_unzipped/res/GK.xml  
  inflating: unlocker_unzipped/res/GQ.xml  
  inflating: unlocker_unzipped/res/GR.xml  
 extracting: unlocker_unzipped/res/Gf.png  
 extracting: unlocker_unzipped/res/Gt.9.png  
  inflating: unlocker_unzipped/res/Gt.xml  
 extracting: unlocker_unzipped/res/H-.png  
  inflating: unlocker_unzipped/res/H4.xml  
  inflating: unlocker_unzipped/res/HC.xml  
  inflating: unlocker_unzipped/res/HQ.xml  
  inflating: unlocker_unzipped/res/Ha.xml  
  inflating: unlocker_unzipped/res/Hd.xml  
  inflating: unlocker_unzipped/res/I3.xml  
 extracting: unlocker_unzipped/res/IX.9.png  
  inflating: unlocker_unzipped/res/Id.xml  
  inflating: unlocker_unzipped/res/In.xml  
  inflating: unlocker_unzipped/res/JC.xml  
  inflating: unlocker_unzipped/res/JD.xml  
  inflating: unlocker_unzipped/res/JD1.xml  
  inflating: unlocker_unzipped/res/JF.xml  
 extracting: unlocker_unzipped/res/JJ.9.png  
  inflating: unlocker_unzipped/res/JQ.xml  
  inflating: unlocker_unzipped/res/JT.xml  
  inflating: unlocker_unzipped/res/JT1.xml  
  inflating: unlocker_unzipped/res/Jl.xml  
  inflating: unlocker_unzipped/res/Jw.xml  
  inflating: unlocker_unzipped/res/K2.xml  
  inflating: unlocker_unzipped/res/K5.xml  
  inflating: unlocker_unzipped/res/K51.xml  
 extracting: unlocker_unzipped/res/KH.9.png  
 extracting: unlocker_unzipped/res/KM.png  
  inflating: unlocker_unzipped/res/KT.xml  
 extracting: unlocker_unzipped/res/K_.9.png  
  inflating: unlocker_unzipped/res/Ke.xml  
  inflating: unlocker_unzipped/res/L-.xml  
  inflating: unlocker_unzipped/res/LI.xml  
  inflating: unlocker_unzipped/res/LT.xml  
  inflating: unlocker_unzipped/res/L_.xml  
  inflating: unlocker_unzipped/res/Lf.xml  
 extracting: unlocker_unzipped/res/Li.9.png  
  inflating: unlocker_unzipped/res/Lt.xml  
  inflating: unlocker_unzipped/res/Lv.xml  
  inflating: unlocker_unzipped/res/M2.xml  
  inflating: unlocker_unzipped/res/M7.xml  
 extracting: unlocker_unzipped/res/MF.9.png  
 extracting: unlocker_unzipped/res/MO.webp  
  inflating: unlocker_unzipped/res/MO.xml  
 extracting: unlocker_unzipped/res/MQ.png  
  inflating: unlocker_unzipped/res/MZ.xml  
  inflating: unlocker_unzipped/res/Mp.xml  
  inflating: unlocker_unzipped/res/N0.xml  
 extracting: unlocker_unzipped/res/NA.9.png  
  inflating: unlocker_unzipped/res/NF.xml  
 extracting: unlocker_unzipped/res/NG.png  
  inflating: unlocker_unzipped/res/NM.xml  
  inflating: unlocker_unzipped/res/NN.xml  
 extracting: unlocker_unzipped/res/NZ.9.png  
 extracting: unlocker_unzipped/res/Nk.9.png  
 extracting: unlocker_unzipped/res/No.9.png  
  inflating: unlocker_unzipped/res/Nu.xml  
  inflating: unlocker_unzipped/res/Ny.xml  
  inflating: unlocker_unzipped/res/OX.xml  
  inflating: unlocker_unzipped/res/Ol.xml  
  inflating: unlocker_unzipped/res/Ox.xml  
  inflating: unlocker_unzipped/res/PQ.xml  
  inflating: unlocker_unzipped/res/PV.xml  
  inflating: unlocker_unzipped/res/PX.xml  
 extracting: unlocker_unzipped/res/Pa.9.png  
 extracting: unlocker_unzipped/res/Pb.png  
 extracting: unlocker_unzipped/res/Pg.9.png  
  inflating: unlocker_unzipped/res/QD.xml  
  inflating: unlocker_unzipped/res/QH.xml  
 extracting: unlocker_unzipped/res/QJ.9.png  
  inflating: unlocker_unzipped/res/QN.xml  
  inflating: unlocker_unzipped/res/QN1.xml  
  inflating: unlocker_unzipped/res/QZ.xml  
  inflating: unlocker_unzipped/res/QZ1.xml  
  inflating: unlocker_unzipped/res/Qd.xml  
  inflating: unlocker_unzipped/res/Qp.xml  
  inflating: unlocker_unzipped/res/Qq.xml  
  inflating: unlocker_unzipped/res/Qr.xml  
  inflating: unlocker_unzipped/res/Qt.xml  
  inflating: unlocker_unzipped/res/Qu.xml  
  inflating: unlocker_unzipped/res/Qy.xml  
  inflating: unlocker_unzipped/res/R2.xml  
  inflating: unlocker_unzipped/res/RH.xml  
  inflating: unlocker_unzipped/res/RL.xml  
  inflating: unlocker_unzipped/res/RM.xml  
 extracting: unlocker_unzipped/res/RV.png  
  inflating: unlocker_unzipped/res/Ro.xml  
  inflating: unlocker_unzipped/res/S6.xml  
  inflating: unlocker_unzipped/res/SG.xml  
 extracting: unlocker_unzipped/res/SV.9.png  
  inflating: unlocker_unzipped/res/Sc.xml  
 extracting: unlocker_unzipped/res/Sn.webp  
  inflating: unlocker_unzipped/res/Sr.xml  
 extracting: unlocker_unzipped/res/Su.9.png  
  inflating: unlocker_unzipped/res/TH.xml  
  inflating: unlocker_unzipped/res/TJ.xml  
  inflating: unlocker_unzipped/res/Tf.xml  
 extracting: unlocker_unzipped/res/Th.png  
 extracting: unlocker_unzipped/res/Tj.9.png  
  inflating: unlocker_unzipped/res/Tn.xml  
  inflating: unlocker_unzipped/res/U0.xml  
  inflating: unlocker_unzipped/res/U8.xml  
 extracting: unlocker_unzipped/res/UE.9.png  
  inflating: unlocker_unzipped/res/UE.xml  
  inflating: unlocker_unzipped/res/UP.xml  
 extracting: unlocker_unzipped/res/UR.png  
  inflating: unlocker_unzipped/res/UX.xml  
  inflating: unlocker_unzipped/res/Uf.xml  
  inflating: unlocker_unzipped/res/V1.xml  
  inflating: unlocker_unzipped/res/V3.xml  
  inflating: unlocker_unzipped/res/V7.xml  
  inflating: unlocker_unzipped/res/VM.xml  
  inflating: unlocker_unzipped/res/VN.xml  
  inflating: unlocker_unzipped/res/VT.xml  
  inflating: unlocker_unzipped/res/Vs.xml  
 extracting: unlocker_unzipped/res/W4.9.png  
  inflating: unlocker_unzipped/res/WC.xml  
  inflating: unlocker_unzipped/res/WK.xml  
  inflating: unlocker_unzipped/res/WP.xml  
 extracting: unlocker_unzipped/res/Wh.png  
 extracting: unlocker_unzipped/res/Wr.png  
 extracting: unlocker_unzipped/res/Wz.png  
 extracting: unlocker_unzipped/res/X3.9.png  
 extracting: unlocker_unzipped/res/X4.9.png  
  inflating: unlocker_unzipped/res/XK.xml  
  inflating: unlocker_unzipped/res/XW.xml  
  inflating: unlocker_unzipped/res/XY.xml  
  inflating: unlocker_unzipped/res/Xf.xml  
  inflating: unlocker_unzipped/res/Xl.xml  
  inflating: unlocker_unzipped/res/Xx.xml  
  inflating: unlocker_unzipped/res/Xz.xml  
 extracting: unlocker_unzipped/res/Y7.9.png  
 extracting: unlocker_unzipped/res/YG.9.png  
  inflating: unlocker_unzipped/res/YN.xml  
  inflating: unlocker_unzipped/res/YW.xml  
  inflating: unlocker_unzipped/res/YW1.xml  
 extracting: unlocker_unzipped/res/Yt.9.png  
 extracting: unlocker_unzipped/res/Yw.9.png  
 extracting: unlocker_unzipped/res/Z8.png  
  inflating: unlocker_unzipped/res/ZC.xml  
  inflating: unlocker_unzipped/res/ZL.xml  
  inflating: unlocker_unzipped/res/ZM.xml  
 extracting: unlocker_unzipped/res/ZN.9.png  
  inflating: unlocker_unzipped/res/ZN.xml  
  inflating: unlocker_unzipped/res/ZW.xml  
  inflating: unlocker_unzipped/res/Zd.xml  
  inflating: unlocker_unzipped/res/Zg.xml  
  inflating: unlocker_unzipped/res/_G.xml  
  inflating: unlocker_unzipped/res/_o.xml  
 extracting: unlocker_unzipped/res/_q.png  
  inflating: unlocker_unzipped/res/a1.xml  
  inflating: unlocker_unzipped/res/a5.xml  
  inflating: unlocker_unzipped/res/aG.xml  
  inflating: unlocker_unzipped/res/aJ.xml  
  inflating: unlocker_unzipped/res/aM.xml  
  inflating: unlocker_unzipped/res/aT.xml  
 extracting: unlocker_unzipped/res/aU.9.png  
  inflating: unlocker_unzipped/res/aW.xml  
  inflating: unlocker_unzipped/res/ai.xml  
 extracting: unlocker_unzipped/res/ar.png  
  inflating: unlocker_unzipped/res/bL.xml  
  inflating: unlocker_unzipped/res/bT.xml  
 extracting: unlocker_unzipped/res/bX.9.png  
  inflating: unlocker_unzipped/res/bb.xml  
  inflating: unlocker_unzipped/res/bm.xml  
  inflating: unlocker_unzipped/res/bt.xml  
  inflating: unlocker_unzipped/res/c0.xml  
  inflating: unlocker_unzipped/res/c2.xml  
  inflating: unlocker_unzipped/res/c5.xml  
  inflating: unlocker_unzipped/res/cA.xml  
  inflating: unlocker_unzipped/res/cA1.xml  
  inflating: unlocker_unzipped/res/cL.xml  
  inflating: unlocker_unzipped/res/cc.xml  
  inflating: unlocker_unzipped/res/cm.xml  
  inflating: unlocker_unzipped/res/color-night-v8/material_timepicker_button_stroke.xml  
  inflating: unlocker_unzipped/res/color-night-v8/material_timepicker_clockface.xml  
  inflating: unlocker_unzipped/res/color-night-v8/material_timepicker_modebutton_tint.xml  
  inflating: unlocker_unzipped/res/color-v23/abc_color_highlight_material.xml  
  inflating: unlocker_unzipped/res/color-v23/abc_tint_btn_checkable.xml  
  inflating: unlocker_unzipped/res/color-v23/abc_tint_default.xml  
  inflating: unlocker_unzipped/res/color-v23/abc_tint_edittext.xml  
  inflating: unlocker_unzipped/res/color-v23/abc_tint_seek_thumb.xml  
  inflating: unlocker_unzipped/res/color-v23/abc_tint_spinner.xml  
  inflating: unlocker_unzipped/res/color-v23/abc_tint_switch_track.xml  
  inflating: unlocker_unzipped/res/color-v31/m3_dynamic_dark_default_color_primary_text.xml  
  inflating: unlocker_unzipped/res/color-v31/m3_dynamic_dark_default_color_secondary_text.xml  
  inflating: unlocker_unzipped/res/color-v31/m3_dynamic_dark_highlighted_text.xml  
  inflating: unlocker_unzipped/res/color-v31/m3_dynamic_dark_hint_foreground.xml  
  inflating: unlocker_unzipped/res/color-v31/m3_dynamic_dark_primary_text_disable_only.xml  
  inflating: unlocker_unzipped/res/color-v31/m3_dynamic_default_color_primary_text.xml  
  inflating: unlocker_unzipped/res/color-v31/m3_dynamic_default_color_secondary_text.xml  
  inflating: unlocker_unzipped/res/color-v31/m3_dynamic_highlighted_text.xml  
  inflating: unlocker_unzipped/res/color-v31/m3_dynamic_hint_foreground.xml  
  inflating: unlocker_unzipped/res/color-v31/m3_dynamic_primary_text_disable_only.xml  
  inflating: unlocker_unzipped/res/color-v31/m3_ref_palette_dynamic_neutral_variant12.xml  
  inflating: unlocker_unzipped/res/color-v31/m3_ref_palette_dynamic_neutral_variant17.xml  
  inflating: unlocker_unzipped/res/color-v31/m3_ref_palette_dynamic_neutral_variant22.xml  
  inflating: unlocker_unzipped/res/color-v31/m3_ref_palette_dynamic_neutral_variant24.xml  
  inflating: unlocker_unzipped/res/color-v31/m3_ref_palette_dynamic_neutral_variant4.xml  
  inflating: unlocker_unzipped/res/color-v31/m3_ref_palette_dynamic_neutral_variant6.xml  
  inflating: unlocker_unzipped/res/color-v31/m3_ref_palette_dynamic_neutral_variant87.xml  
  inflating: unlocker_unzipped/res/color-v31/m3_ref_palette_dynamic_neutral_variant92.xml  
  inflating: unlocker_unzipped/res/color-v31/m3_ref_palette_dynamic_neutral_variant94.xml  
  inflating: unlocker_unzipped/res/color-v31/m3_ref_palette_dynamic_neutral_variant96.xml  
  inflating: unlocker_unzipped/res/color-v31/m3_ref_palette_dynamic_neutral_variant98.xml  
  inflating: unlocker_unzipped/res/color/abc_background_cache_hint_selector_material_dark.xml  
  inflating: unlocker_unzipped/res/color/abc_background_cache_hint_selector_material_light.xml  
  inflating: unlocker_unzipped/res/color/abc_hint_foreground_material_dark.xml  
  inflating: unlocker_unzipped/res/color/abc_hint_foreground_material_light.xml  
  inflating: unlocker_unzipped/res/color/abc_primary_text_disable_only_material_dark.xml  
  inflating: unlocker_unzipped/res/color/abc_primary_text_disable_only_material_light.xml  
  inflating: unlocker_unzipped/res/color/abc_primary_text_material_dark.xml  
  inflating: unlocker_unzipped/res/color/abc_primary_text_material_light.xml  
  inflating: unlocker_unzipped/res/color/abc_search_url_text.xml  
  inflating: unlocker_unzipped/res/color/abc_secondary_text_material_dark.xml  
  inflating: unlocker_unzipped/res/color/abc_secondary_text_material_light.xml  
  inflating: unlocker_unzipped/res/color/design_box_stroke_color.xml  
  inflating: unlocker_unzipped/res/color/design_error.xml  
  inflating: unlocker_unzipped/res/color/design_icon_tint.xml  
  inflating: unlocker_unzipped/res/color/m3_assist_chip_icon_tint_color.xml  
  inflating: unlocker_unzipped/res/color/m3_bottom_sheet_drag_handle_color.xml  
  inflating: unlocker_unzipped/res/color/m3_button_background_color_selector.xml  
  inflating: unlocker_unzipped/res/color/m3_button_foreground_color_selector.xml  
  inflating: unlocker_unzipped/res/color/m3_button_outline_color_selector.xml  
  inflating: unlocker_unzipped/res/color/m3_button_ripple_color_selector.xml  
  inflating: unlocker_unzipped/res/color/m3_calendar_item_disabled_text.xml  
  inflating: unlocker_unzipped/res/color/m3_calendar_item_stroke_color.xml  
  inflating: unlocker_unzipped/res/color/m3_card_foreground_color.xml  
  inflating: unlocker_unzipped/res/color/m3_card_ripple_color.xml  
  inflating: unlocker_unzipped/res/color/m3_card_stroke_color.xml  
  inflating: unlocker_unzipped/res/color/m3_checkbox_button_icon_tint.xml  
  inflating: unlocker_unzipped/res/color/m3_checkbox_button_tint.xml  
  inflating: unlocker_unzipped/res/color/m3_chip_assist_text_color.xml  
  inflating: unlocker_unzipped/res/color/m3_chip_background_color.xml  
  inflating: unlocker_unzipped/res/color/m3_chip_ripple_color.xml  
  inflating: unlocker_unzipped/res/color/m3_chip_stroke_color.xml  
  inflating: unlocker_unzipped/res/color/m3_chip_text_color.xml  
  inflating: unlocker_unzipped/res/color/m3_dark_default_color_primary_text.xml  
  inflating: unlocker_unzipped/res/color/m3_dark_default_color_secondary_text.xml  
  inflating: unlocker_unzipped/res/color/m3_dark_highlighted_text.xml  
  inflating: unlocker_unzipped/res/color/m3_dark_hint_foreground.xml  
  inflating: unlocker_unzipped/res/color/m3_dark_primary_text_disable_only.xml  
  inflating: unlocker_unzipped/res/color/m3_default_color_primary_text.xml  
  inflating: unlocker_unzipped/res/color/m3_default_color_secondary_text.xml  
  inflating: unlocker_unzipped/res/color/m3_efab_ripple_color_selector.xml  
  inflating: unlocker_unzipped/res/color/m3_fab_efab_background_color_selector.xml  
  inflating: unlocker_unzipped/res/color/m3_fab_efab_foreground_color_selector.xml  
  inflating: unlocker_unzipped/res/color/m3_fab_ripple_color_selector.xml  
  inflating: unlocker_unzipped/res/color/m3_filled_icon_button_container_color_selector.xml  
  inflating: unlocker_unzipped/res/color/m3_highlighted_text.xml  
  inflating: unlocker_unzipped/res/color/m3_hint_foreground.xml  
  inflating: unlocker_unzipped/res/color/m3_icon_button_icon_color_selector.xml  
  inflating: unlocker_unzipped/res/color/m3_navigation_bar_item_with_indicator_icon_tint.xml  
  inflating: unlocker_unzipped/res/color/m3_navigation_bar_item_with_indicator_label_tint.xml  
  inflating: unlocker_unzipped/res/color/m3_navigation_bar_ripple_color_selector.xml  
  inflating: unlocker_unzipped/res/color/m3_navigation_item_background_color.xml  
  inflating: unlocker_unzipped/res/color/m3_navigation_item_icon_tint.xml  
  inflating: unlocker_unzipped/res/color/m3_navigation_item_ripple_color.xml  
  inflating: unlocker_unzipped/res/color/m3_navigation_item_text_color.xml  
  inflating: unlocker_unzipped/res/color/m3_navigation_rail_item_with_indicator_icon_tint.xml  
  inflating: unlocker_unzipped/res/color/m3_navigation_rail_item_with_indicator_label_tint.xml  
  inflating: unlocker_unzipped/res/color/m3_navigation_rail_ripple_color_selector.xml  
  inflating: unlocker_unzipped/res/color/m3_primary_text_disable_only.xml  
  inflating: unlocker_unzipped/res/color/m3_radiobutton_button_tint.xml  
  inflating: unlocker_unzipped/res/color/m3_radiobutton_ripple_tint.xml  
  inflating: unlocker_unzipped/res/color/m3_selection_control_ripple_color_selector.xml  
  inflating: unlocker_unzipped/res/color/m3_simple_item_ripple_color.xml  
  inflating: unlocker_unzipped/res/color/m3_slider_active_tick_marks_color.xml  
  inflating: unlocker_unzipped/res/color/m3_slider_active_track_color.xml  
  inflating: unlocker_unzipped/res/color/m3_slider_inactive_tick_marks_color.xml  
  inflating: unlocker_unzipped/res/color/m3_slider_inactive_track_color.xml  
  inflating: unlocker_unzipped/res/color/m3_slider_thumb_color.xml  
  inflating: unlocker_unzipped/res/color/m3_standard_toolbar_button_text_color_selector.xml  
  inflating: unlocker_unzipped/res/color/m3_standard_toolbar_icon_button_container_color_selector.xml  
  inflating: unlocker_unzipped/res/color/m3_standard_toolbar_icon_button_icon_color_selector.xml  
  inflating: unlocker_unzipped/res/color/m3_standard_toolbar_icon_button_ripple_color_selector.xml  
  inflating: unlocker_unzipped/res/color/m3_switch_thumb_tint.xml  
  inflating: unlocker_unzipped/res/color/m3_switch_track_tint.xml  
  inflating: unlocker_unzipped/res/color/m3_tabs_icon_color.xml  
  inflating: unlocker_unzipped/res/color/m3_tabs_icon_color_secondary.xml  
  inflating: unlocker_unzipped/res/color/m3_tabs_ripple_color.xml  
  inflating: unlocker_unzipped/res/color/m3_tabs_ripple_color_secondary.xml  
  inflating: unlocker_unzipped/res/color/m3_tabs_text_color.xml  
  inflating: unlocker_unzipped/res/color/m3_tabs_text_color_secondary.xml  
  inflating: unlocker_unzipped/res/color/m3_text_button_background_color_selector.xml  
  inflating: unlocker_unzipped/res/color/m3_text_button_foreground_color_selector.xml  
  inflating: unlocker_unzipped/res/color/m3_text_button_ripple_color_selector.xml  
  inflating: unlocker_unzipped/res/color/m3_textfield_filled_background_color.xml  
  inflating: unlocker_unzipped/res/color/m3_textfield_indicator_text_color.xml  
  inflating: unlocker_unzipped/res/color/m3_textfield_input_text_color.xml  
  inflating: unlocker_unzipped/res/color/m3_textfield_label_color.xml  
  inflating: unlocker_unzipped/res/color/m3_textfield_stroke_color.xml  
  inflating: unlocker_unzipped/res/color/m3_timepicker_button_background_color.xml  
  inflating: unlocker_unzipped/res/color/m3_timepicker_button_ripple_color.xml  
  inflating: unlocker_unzipped/res/color/m3_timepicker_button_text_color.xml  
  inflating: unlocker_unzipped/res/color/m3_timepicker_clock_text_color.xml  
  inflating: unlocker_unzipped/res/color/m3_timepicker_display_background_color.xml  
  inflating: unlocker_unzipped/res/color/m3_timepicker_display_ripple_color.xml  
  inflating: unlocker_unzipped/res/color/m3_timepicker_display_text_color.xml  
  inflating: unlocker_unzipped/res/color/m3_timepicker_secondary_text_button_ripple_color.xml  
  inflating: unlocker_unzipped/res/color/m3_timepicker_secondary_text_button_text_color.xml  
  inflating: unlocker_unzipped/res/color/m3_timepicker_time_input_stroke_color.xml  
  inflating: unlocker_unzipped/res/color/m3_tonal_button_ripple_color_selector.xml  
  inflating: unlocker_unzipped/res/color/m3_vibrant_toolbar_button_text_color_selector.xml  
  inflating: unlocker_unzipped/res/color/m3_vibrant_toolbar_icon_button_container_color_selector.xml  
  inflating: unlocker_unzipped/res/color/m3_vibrant_toolbar_icon_button_icon_color_selector.xml  
  inflating: unlocker_unzipped/res/color/m3_vibrant_toolbar_icon_button_ripple_color_selector.xml  
  inflating: unlocker_unzipped/res/color/material_divider_color.xml  
  inflating: unlocker_unzipped/res/color/material_on_surface_disabled.xml  
  inflating: unlocker_unzipped/res/color/material_on_surface_emphasis_high_type.xml  
  inflating: unlocker_unzipped/res/color/material_on_surface_emphasis_medium.xml  
  inflating: unlocker_unzipped/res/color/material_slider_active_tick_marks_color.xml  
  inflating: unlocker_unzipped/res/color/material_slider_active_track_color.xml  
  inflating: unlocker_unzipped/res/color/material_slider_halo_color.xml  
  inflating: unlocker_unzipped/res/color/material_slider_inactive_tick_marks_color.xml  
  inflating: unlocker_unzipped/res/color/material_slider_inactive_track_color.xml  
  inflating: unlocker_unzipped/res/color/material_slider_thumb_color.xml  
  inflating: unlocker_unzipped/res/color/material_timepicker_button_background.xml  
  inflating: unlocker_unzipped/res/color/material_timepicker_button_stroke.xml  
  inflating: unlocker_unzipped/res/color/material_timepicker_clock_text_color.xml  
  inflating: unlocker_unzipped/res/color/material_timepicker_clockface.xml  
  inflating: unlocker_unzipped/res/color/material_timepicker_modebutton_tint.xml  
  inflating: unlocker_unzipped/res/color/mtrl_btn_bg_color_selector.xml  
  inflating: unlocker_unzipped/res/color/mtrl_btn_ripple_color.xml  
  inflating: unlocker_unzipped/res/color/mtrl_btn_stroke_color_selector.xml  
  inflating: unlocker_unzipped/res/color/mtrl_btn_text_btn_bg_color_selector.xml  
  inflating: unlocker_unzipped/res/color/mtrl_btn_text_btn_ripple_color.xml  
  inflating: unlocker_unzipped/res/color/mtrl_btn_text_color_selector.xml  
  inflating: unlocker_unzipped/res/color/mtrl_calendar_item_stroke_color.xml  
  inflating: unlocker_unzipped/res/color/mtrl_calendar_selected_range.xml  
  inflating: unlocker_unzipped/res/color/mtrl_card_view_foreground.xml  
  inflating: unlocker_unzipped/res/color/mtrl_card_view_ripple.xml  
  inflating: unlocker_unzipped/res/color/mtrl_chip_background_color.xml  
  inflating: unlocker_unzipped/res/color/mtrl_chip_close_icon_tint.xml  
  inflating: unlocker_unzipped/res/color/mtrl_chip_surface_color.xml  
  inflating: unlocker_unzipped/res/color/mtrl_chip_text_color.xml  
  inflating: unlocker_unzipped/res/color/mtrl_choice_chip_background_color.xml  
  inflating: unlocker_unzipped/res/color/mtrl_choice_chip_ripple_color.xml  
  inflating: unlocker_unzipped/res/color/mtrl_choice_chip_text_color.xml  
  inflating: unlocker_unzipped/res/color/mtrl_error.xml  
  inflating: unlocker_unzipped/res/color/mtrl_fab_bg_color_selector.xml  
  inflating: unlocker_unzipped/res/color/mtrl_fab_icon_text_color_selector.xml  
  inflating: unlocker_unzipped/res/color/mtrl_fab_ripple_color.xml  
  inflating: unlocker_unzipped/res/color/mtrl_filled_background_color.xml  
  inflating: unlocker_unzipped/res/color/mtrl_filled_icon_tint.xml  
  inflating: unlocker_unzipped/res/color/mtrl_filled_stroke_color.xml  
  inflating: unlocker_unzipped/res/color/mtrl_indicator_text_color.xml  
  inflating: unlocker_unzipped/res/color/mtrl_navigation_bar_item_tint.xml  
  inflating: unlocker_unzipped/res/color/mtrl_navigation_bar_ripple_color.xml  
  inflating: unlocker_unzipped/res/color/mtrl_navigation_item_background_color.xml  
  inflating: unlocker_unzipped/res/color/mtrl_navigation_item_icon_tint.xml  
  inflating: unlocker_unzipped/res/color/mtrl_navigation_item_text_color.xml  
  inflating: unlocker_unzipped/res/color/mtrl_on_primary_text_btn_text_color_selector.xml  
  inflating: unlocker_unzipped/res/color/mtrl_on_surface_ripple_color.xml  
  inflating: unlocker_unzipped/res/color/mtrl_outlined_icon_tint.xml  
  inflating: unlocker_unzipped/res/color/mtrl_outlined_stroke_color.xml  
  inflating: unlocker_unzipped/res/color/mtrl_popupmenu_overlay_color.xml  
  inflating: unlocker_unzipped/res/color/mtrl_switch_thumb_icon_tint.xml  
  inflating: unlocker_unzipped/res/color/mtrl_switch_thumb_tint.xml  
  inflating: unlocker_unzipped/res/color/mtrl_switch_track_decoration_tint.xml  
  inflating: unlocker_unzipped/res/color/mtrl_switch_track_tint.xml  
  inflating: unlocker_unzipped/res/color/mtrl_tabs_icon_color_selector.xml  
  inflating: unlocker_unzipped/res/color/mtrl_tabs_legacy_text_color_selector.xml  
  inflating: unlocker_unzipped/res/color/mtrl_tabs_ripple_color.xml  
  inflating: unlocker_unzipped/res/color/mtrl_text_btn_text_color_selector.xml  
  inflating: unlocker_unzipped/res/color/switch_thumb_material_dark.xml  
  inflating: unlocker_unzipped/res/color/switch_thumb_material_light.xml  
  inflating: unlocker_unzipped/res/cs.xml  
  inflating: unlocker_unzipped/res/cy.xml  
 extracting: unlocker_unzipped/res/d2.webp  
  inflating: unlocker_unzipped/res/d2.xml  
 extracting: unlocker_unzipped/res/d3.png  
 extracting: unlocker_unzipped/res/d5.9.png  
  inflating: unlocker_unzipped/res/dC.xml  
  inflating: unlocker_unzipped/res/dO.xml  
  inflating: unlocker_unzipped/res/dS.xml  
 extracting: unlocker_unzipped/res/dW.png  
 extracting: unlocker_unzipped/res/dY.png  
  inflating: unlocker_unzipped/res/dj.xml  
  inflating: unlocker_unzipped/res/dw.xml  
  inflating: unlocker_unzipped/res/e0.xml  
  inflating: unlocker_unzipped/res/eA.xml  
  inflating: unlocker_unzipped/res/eH.xml  
  inflating: unlocker_unzipped/res/eH1.xml  
  inflating: unlocker_unzipped/res/eK.xml  
  inflating: unlocker_unzipped/res/eM.xml  
 extracting: unlocker_unzipped/res/eR.png  
 extracting: unlocker_unzipped/res/eT.9.png  
  inflating: unlocker_unzipped/res/eW.xml  
  inflating: unlocker_unzipped/res/eZ.xml  
 extracting: unlocker_unzipped/res/ej.9.png  
  inflating: unlocker_unzipped/res/ej.xml  
 extracting: unlocker_unzipped/res/ev.9.png  
  inflating: unlocker_unzipped/res/f6.xml  
 extracting: unlocker_unzipped/res/fM.9.png  
  inflating: unlocker_unzipped/res/fW.xml  
  inflating: unlocker_unzipped/res/f_.xml  
  inflating: unlocker_unzipped/res/fd.xml  
  inflating: unlocker_unzipped/res/fg.xml  
  inflating: unlocker_unzipped/res/fp.xml  
 extracting: unlocker_unzipped/res/fq.webp  
 extracting: unlocker_unzipped/res/g-.png  
  inflating: unlocker_unzipped/res/g3.xml  
  inflating: unlocker_unzipped/res/gC.xml  
  inflating: unlocker_unzipped/res/gD.xml  
  inflating: unlocker_unzipped/res/gG.xml  
  inflating: unlocker_unzipped/res/gR.xml  
 extracting: unlocker_unzipped/res/gZ.9.png  
 extracting: unlocker_unzipped/res/gj.9.png  
 extracting: unlocker_unzipped/res/gt.9.png  
  inflating: unlocker_unzipped/res/h4.xml  
 extracting: unlocker_unzipped/res/h7.9.png  
  inflating: unlocker_unzipped/res/hP.xml  
  inflating: unlocker_unzipped/res/hP1.xml  
 extracting: unlocker_unzipped/res/hZ.9.png  
  inflating: unlocker_unzipped/res/hb.xml  
  inflating: unlocker_unzipped/res/hc.xml  
 extracting: unlocker_unzipped/res/hh.9.png  
  inflating: unlocker_unzipped/res/hq.xml  
  inflating: unlocker_unzipped/res/hu.xml  
  inflating: unlocker_unzipped/res/hv.xml  
 extracting: unlocker_unzipped/res/i6.9.png  
  inflating: unlocker_unzipped/res/iI.xml  
 extracting: unlocker_unzipped/res/iO.png  
 extracting: unlocker_unzipped/res/iR.9.png  
  inflating: unlocker_unzipped/res/iZ.xml  
 extracting: unlocker_unzipped/res/io.9.png  
  inflating: unlocker_unzipped/res/j3.xml  
 extracting: unlocker_unzipped/res/j4.png  
 extracting: unlocker_unzipped/res/jS.9.png  
 extracting: unlocker_unzipped/res/jS1.9.png  
 extracting: unlocker_unzipped/res/jW.png  
 extracting: unlocker_unzipped/res/j_.webp  
  inflating: unlocker_unzipped/res/k8.xml  
  inflating: unlocker_unzipped/res/k9.xml  
 extracting: unlocker_unzipped/res/kJ.9.png  
  inflating: unlocker_unzipped/res/kN.xml  
  inflating: unlocker_unzipped/res/k_.xml  
  inflating: unlocker_unzipped/res/kg.xml  
  inflating: unlocker_unzipped/res/kh.xml  
  inflating: unlocker_unzipped/res/kj.xml  
  inflating: unlocker_unzipped/res/kj1.xml  
  inflating: unlocker_unzipped/res/kn.xml  
 extracting: unlocker_unzipped/res/kp.png  
  inflating: unlocker_unzipped/res/lN.xml  
 extracting: unlocker_unzipped/res/lP.9.png  
 extracting: unlocker_unzipped/res/ly.png  
 extracting: unlocker_unzipped/res/m0.png  
  inflating: unlocker_unzipped/res/mA.xml  
 extracting: unlocker_unzipped/res/mm.9.png  
  inflating: unlocker_unzipped/res/mx.xml  
 extracting: unlocker_unzipped/res/nI.9.png  
  inflating: unlocker_unzipped/res/nP.xml  
  inflating: unlocker_unzipped/res/nT.xml  
 extracting: unlocker_unzipped/res/nf.png  
  inflating: unlocker_unzipped/res/nl.xml  
  inflating: unlocker_unzipped/res/nm.xml  
 extracting: unlocker_unzipped/res/o9.9.png  
  inflating: unlocker_unzipped/res/oP.xml  
  inflating: unlocker_unzipped/res/oY.xml  
 extracting: unlocker_unzipped/res/o_.png  
  inflating: unlocker_unzipped/res/oa.xml  
 extracting: unlocker_unzipped/res/op.9.png  
  inflating: unlocker_unzipped/res/p0.xml  
  inflating: unlocker_unzipped/res/pF.xml  
  inflating: unlocker_unzipped/res/pU.xml  
 extracting: unlocker_unzipped/res/pk.png  
  inflating: unlocker_unzipped/res/pn.xml  
 extracting: unlocker_unzipped/res/ps.9.png  
  inflating: unlocker_unzipped/res/pw.xml  
 extracting: unlocker_unzipped/res/py.9.png  
  inflating: unlocker_unzipped/res/q6.xml  
  inflating: unlocker_unzipped/res/qA.xml  
 extracting: unlocker_unzipped/res/qD.9.png  
 extracting: unlocker_unzipped/res/qp.png  
 extracting: unlocker_unzipped/res/qs.webp  
  inflating: unlocker_unzipped/res/qx.xml  
  inflating: unlocker_unzipped/res/qz.xml  
  inflating: unlocker_unzipped/res/rI.xml  
  inflating: unlocker_unzipped/res/rW.xml  
  inflating: unlocker_unzipped/res/rY.xml  
  inflating: unlocker_unzipped/res/rd.xml  
 extracting: unlocker_unzipped/res/rj.9.png  
  inflating: unlocker_unzipped/res/rx.xml  
 extracting: unlocker_unzipped/res/s0.png  
 extracting: unlocker_unzipped/res/s3.9.png  
  inflating: unlocker_unzipped/res/sA.xml  
 extracting: unlocker_unzipped/res/sK.webp  
  inflating: unlocker_unzipped/res/sO.xml  
  inflating: unlocker_unzipped/res/sX.xml  
 extracting: unlocker_unzipped/res/sg.9.png  
  inflating: unlocker_unzipped/res/sl.xml  
  inflating: unlocker_unzipped/res/sn.xml  
  inflating: unlocker_unzipped/res/t8.xml  
 extracting: unlocker_unzipped/res/tG.png  
  inflating: unlocker_unzipped/res/tI.xml  
  inflating: unlocker_unzipped/res/tL.xml  
 extracting: unlocker_unzipped/res/tS.png  
  inflating: unlocker_unzipped/res/tS.xml  
 extracting: unlocker_unzipped/res/tU.9.png  
 extracting: unlocker_unzipped/res/tZ.9.png  
 extracting: unlocker_unzipped/res/te.png  
  inflating: unlocker_unzipped/res/tp.xml  
  inflating: unlocker_unzipped/res/u0.xml  
 extracting: unlocker_unzipped/res/u3.png  
 extracting: unlocker_unzipped/res/u5.webp  
  inflating: unlocker_unzipped/res/uJ.xml  
 extracting: unlocker_unzipped/res/uL.9.png  
  inflating: unlocker_unzipped/res/uR.xml  
 extracting: unlocker_unzipped/res/uj.9.png  
  inflating: unlocker_unzipped/res/up.xml  
 extracting: unlocker_unzipped/res/ut.9.png  
 extracting: unlocker_unzipped/res/uu.9.png  
  inflating: unlocker_unzipped/res/v-.xml  
 extracting: unlocker_unzipped/res/v4.9.png  
  inflating: unlocker_unzipped/res/v9.xml  
  inflating: unlocker_unzipped/res/vG.xml  
  inflating: unlocker_unzipped/res/vH.xml  
  inflating: unlocker_unzipped/res/vJ.xml  
  inflating: unlocker_unzipped/res/vT.xml  
  inflating: unlocker_unzipped/res/vZ.xml  
  inflating: unlocker_unzipped/res/vf.xml  
  inflating: unlocker_unzipped/res/vl.xml  
  inflating: unlocker_unzipped/res/vn.xml  
  inflating: unlocker_unzipped/res/vq.xml  
 extracting: unlocker_unzipped/res/vz.9.png  
  inflating: unlocker_unzipped/res/w9.xml  
 extracting: unlocker_unzipped/res/wL.9.png  
  inflating: unlocker_unzipped/res/wP.xml  
 extracting: unlocker_unzipped/res/w_.png  
 extracting: unlocker_unzipped/res/x3.9.png  
 extracting: unlocker_unzipped/res/xH.png  
 extracting: unlocker_unzipped/res/xR.9.png  
  inflating: unlocker_unzipped/res/xa.xml  
  inflating: unlocker_unzipped/res/xd.xml  
  inflating: unlocker_unzipped/res/xj.xml  
 extracting: unlocker_unzipped/res/yH.9.png  
  inflating: unlocker_unzipped/res/yS.xml  
  inflating: unlocker_unzipped/res/yT.xml  
  inflating: unlocker_unzipped/res/yV.xml  
 extracting: unlocker_unzipped/res/yY.9.png  
  inflating: unlocker_unzipped/res/ya.xml  
 extracting: unlocker_unzipped/res/yg.9.png  
 extracting: unlocker_unzipped/res/yw.webp  
 extracting: unlocker_unzipped/res/z9.9.png  
 extracting: unlocker_unzipped/res/zE.png  
  inflating: unlocker_unzipped/res/zG.xml  
 extracting: unlocker_unzipped/res/zV.9.png  
  inflating: unlocker_unzipped/res/zc.xml  
  inflating: unlocker_unzipped/res/zq.xml  
 extracting: unlocker_unzipped/resources.arsc  
lib
├── arm64-v8a
│   └── libnativelib.so
├── armeabi-v7a
│   └── libnativelib.so
├── x86
│   └── libnativelib.so
└── x86_64
    └── libnativelib.so

5 directories, 4 files
```

Yes, it exists. Let's load the libnativelib.so file into Ghidra to locate **generateInsecureOtp**.
After finding the **generateInsecureOtp** function, you will get something like this:


```
undefined8
Java_com_svattt_nativelib_NativeLib_generateInsecureOtp
          (long *param_1,undefined8 param_2,int param_3)

{
  undefined8 uVar1;
  undefined1 *puVar2;
  long in_FS_OFFSET;
  native_otp local_38;
  undefined1 local_37 [15];
  undefined1 *local_28;
  long local_20;
  
  local_20 = *(long *)(in_FS_OFFSET + 0x28);
                    /* try { // try from 001214da to 001214e6 has its CatchHandler @ 0012154d */
  native_otp::insecure_otp_from_seed(&local_38,0x1350191,param_3);
  puVar2 = local_28;
  if (((byte)local_38 & 1) == 0) {
    puVar2 = local_37;
  }
                    /* try { // try from 001214fc to 00121504 has its CatchHandler @ 00121535 */
  uVar1 = (**(code **)(*param_1 + 0x538))(param_1,puVar2);
  if (((byte)local_38 & 1) != 0) {
    operator.delete(local_28);
  }
  if (*(long *)(in_FS_OFFSET + 0x28) != local_20) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return uVar1;
}

This function call:
 
native_otp::insecure_otp_from_seed(&local_38, 0x1350191, param_3);
```
=> **generateInsecureOtp(15)** takes an OTP from address 0x1350191 with length 15.
So we need to go into the function **native_otp::insecure_otp_from_seed** to see how the program creates the OTP.

```
/* native_otp::insecure_otp_from_seed(unsigned long, int) */

native_otp * __thiscall
native_otp::insecure_otp_from_seed(native_otp *this,ulong param_1,int param_2)

{
  char *pcVar1;
  logic_error *this_00;
  long lVar2;
  long lVar3;
  ulong uVar4;
  long lVar5;
  long in_FS_OFFSET;
  undefined8 local_a08;
  ulong local_a00;
  ulong local_9f8 [314];
  
  local_9f8[0x139] = *(long *)(in_FS_OFFSET + 0x28);
  pcVar1 = (char *)operator.new(0x40);
  builtin_strncpy(pcVar1,"0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz",0x3f);
  if (param_2 - 0x41U < 0xffffffc0) {
    this_00 = (logic_error *)__cxa_allocate_exception(0x10);
                    /* try { // try from 001213af to 001213bd has its CatchHandler @ 001213fa */
    std::logic_error::logic_error(this_00,"length out of range");
    *(undefined ***)this_00 = &PTR_~invalid_argument_0014a188;
    if (*(long *)(in_FS_OFFSET + 0x28) == local_9f8[0x139]) {
                    /* try { // try from 001213df to 001213f4 has its CatchHandler @ 001213f5 */
                    /* WARNING: Subroutine does not return */
      __cxa_throw(this_00,&std::invalid_argument::typeinfo,std::invalid_argument::~invalid_argumen t)
      ;
    }
  }
  else {
    lVar2 = 1;
    lVar3 = 2;
    uVar4 = param_1;
    while( true ) {
      uVar4 = ((uVar4 >> 0x3e ^ uVar4) * 0x5851f42d4c957f2d + lVar3) - 1;
      (&local_a00)[lVar3] = uVar4;
      if (lVar3 == 0x138) break;
      lVar5 = (uVar4 >> 0x3e ^ uVar4) * 0x5851f42d4c957f2d;
      uVar4 = lVar2 + 1 + lVar5;
      local_9f8[lVar3] = lVar5 + lVar3;
      lVar2 = lVar2 + 2;
      lVar3 = lVar3 + 2;
    }
    local_9f8[0x138] = 0;
    local_a08 = 0;
    local_a00 = 0x3d;
    *(undefined8 *)this = 0;
    *(undefined8 *)(this + 8) = 0;
    *(undefined8 *)(this + 0x10) = 0;
    local_9f8[0] = param_1;
                    /* try { // try from 00121338 to 0012133f has its CatchHandler @ 00121407 */
    std::__ndk1::basic_string<>::reserve((basic_string<> *)this,(ulong)(uint)param_2);
    if (0 < param_2) {
      do {
                    /* try { // try from 00121350 to 00121369 has its CatchHandler @ 00121409 */
        uVar4 = std::__ndk1::uniform_int_distribution<>::operator()
                          ((uniform_int_distribution<> *)&local_a08,
                           (mersenne_twister_engine *)local_9f8,(param_type *)&local_a08);
        std::__ndk1::basic_string<>::push_back((basic_string<> *)this,pcVar1[uVar4]);
        param_2 = param_2 + -1;
      } while (param_2 != 0);
    }
    operator.delete(pcVar1);
    if (*(long *)(in_FS_OFFSET + 0x28) == local_9f8[0x139]) {
      return this;
    }
  }
                    /* WARNING: Subroutine does not return */
  __stack_chk_fail();
}
```

**insecure_otp_from_seed** usage:

+ std::mt19937_64 generate random number

+ Alphabet: "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"

+ Create 15-digit from uniform_int_distribution(0, 61) then index to alphabet.

Combine with the previous function, I wrote a python code to generate the passkey:

```
W = 64
N = 312
M = 156
R = 31

A = 0xB5026F5AA96619E9
U = 29
D = 0x5555555555555555
S = 17
B = 0x71D67FFFEDA60000
T = 37
C = 0xFFF7EEE000000000
L = 43
F = 6364136223846793005

LOWER_MASK = (1 << R) - 1
UPPER_MASK = ((1 << W) - 1) & (~LOWER_MASK)
MASK       = (1 << W) - 1

ALPHABET = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"  


class MT19937_64:
    def __init__(self, seed: int):
        self.mt = [0] * N
        self.index = N
        self.seed(seed)

    def seed(self, seed: int):
        self.mt[0] = seed & MASK
        for i in range(1, N):
            self.mt[i] = (F * (self.mt[i-1] ^ (self.mt[i-1] >> (W - 2))) + i) & MASK
        self.index = N

    def twist(self):
        for i in range(N):
            x  = (self.mt[i] & UPPER_MASK) + (self.mt[(i + 1) % N] & LOWER_MASK)
            xA = x >> 1
            if x & 1:
                xA ^= A
            self.mt[i] = (self.mt[(i + M) % N] ^ xA) & MASK
        self.index = 0

    def rand_u64(self) -> int:
        if self.index >= N:
            self.twist()

        x = self.mt[self.index]
        self.index += 1

        y = x ^ ((x >> U) & D)
        y ^= (y << S) & B
        y ^= (y << T) & C
        y ^= (y >> L)
        return y & MASK


def generate_passkey(seed: int, length: int) -> str:
    rng = MT19937_64(seed)
    out = []
    for _ in range(length):
        while True:
            x = rng.rand_u64()
            idx = x & 0x3F       
            if idx < 62: 
                out.append(ALPHABET[idx])
                break
    return "".join(out)


if __name__ == "__main__":
    SEED   = 0x1350191   
    LENGTH = 15           
    passkey = generate_passkey(SEED, LENGTH)
    print(passkey)
```

Output:

```
pjEmGyPhO0U441E
```

Convert into SHA-1:

![image](https://hackmd.io/_uploads/Bypq6lFlbl.png)


=> Flag is: **CSCV2025{e7b145e535cc978adcf119bb8ee27824e4c13c12}**

**3) Hello World**

**+Decription:**
![image](https://hackmd.io/_uploads/SJB_0xKgZg.png)

**+Analyze:**
Basic information about this file:
![image](https://hackmd.io/_uploads/HkbkybYgZl.png)
I could not run this file in my device so i put this file in IDA to decompile this file:
```
NTSTATUS __stdcall DriverEntry(PDRIVER_OBJECT DriverObject, PUNICODE_STRING RegistryPath)
{
  unsigned __int16 v2; // bx
  __int64 v3; // r15
  __int64 v4; // rax
  wchar_t v5; // dx
  __int64 v6; // rax
  __int64 v7; // rcx
  __int64 v8; // rax
  __int64 v9; // rax
  __int64 v10; // rcx
  unsigned __int16 i; // dx
  __int64 v12; // rbx
  struct _UNICODE_STRING v14; // [rsp+60h] [rbp-A0h] BYREF
  struct _UNICODE_STRING DisplayString; // [rsp+70h] [rbp-90h] BYREF
  char Buffer[2]; // [rsp+80h] [rbp-80h] BYREF
  unsigned __int16 v17; // [rsp+82h] [rbp-7Eh]
  __int16 v18; // [rsp+84h] [rbp-7Ch]
  union _LARGE_INTEGER ByteOffset; // [rsp+90h] [rbp-70h] BYREF
  int v20; // [rsp+98h] [rbp-68h] BYREF
  const wchar_t *v21; // [rsp+A0h] [rbp-60h]
  struct _UNICODE_STRING v22; // [rsp+A8h] [rbp-58h] BYREF
  struct _OBJECT_ATTRIBUTES ObjectAttributes; // [rsp+B8h] [rbp-48h] BYREF
  struct _IO_STATUS_BLOCK IoStatusBlock; // [rsp+E8h] [rbp-18h] BYREF
  wchar_t Dst[792]; // [rsp+100h] [rbp+0h] BYREF
  void *EventHandle; // [rsp+748h] [rbp+648h] BYREF
  void *FileHandle; // [rsp+750h] [rbp+650h] BYREF
  LARGE_INTEGER Interval; // [rsp+758h] [rbp+658h] BYREF

  *(_DWORD *)&DisplayString.Length = 7995512;
  DisplayString.Buffer = L"!!! STOP !!!\n\nThe machine was protected by an application.\n\n";
  v2 = 0;
  NtDisplayString(&DisplayString);
  v21 = L"\\Device\\KeyboardClass0";
  ObjectAttributes.Attributes = 64;
  ObjectAttributes.ObjectName = (PUNICODE_STRING)&v20;
  v20 = 3014700;
  ObjectAttributes.Length = 48;
  ObjectAttributes.RootDirectory = 0i64;
  *(_OWORD *)&ObjectAttributes.SecurityDescriptor = 0i64;
  NtCreateFile(&FileHandle, 0x80100080, &ObjectAttributes, &IoStatusBlock, 0i64, 0x80u, 0, 1u, 1u, 0i64, 0);
  ObjectAttributes.Length = 48;
  memset(&ObjectAttributes.RootDirectory, 0, 20);
  *(_OWORD *)&ObjectAttributes.SecurityDescriptor = 0i64;
  NtCreateEvent(&EventHandle, 0x1F0003u, &ObjectAttributes, NotificationEvent, 0);
  Interval.QuadPart = -100000i64;
  DisplayString.Buffer = L"Enter the password:\n";
  *(_DWORD *)&DisplayString.Length = 2752552;
  NtDisplayString(&DisplayString);
  v3 = 0xBFFBFFDFFF3FFCi64;
  while ( 1 )
  {
    NtReadFile(FileHandle, EventHandle, 0i64, 0i64, &IoStatusBlock, Buffer, 0xCu, &ByteOffset, 0i64);
    NtWaitForSingleObject(EventHandle, 1u, 0i64);
    if ( v18 )
      goto LABEL_36;
    if ( v17 == 28 )
      break;
    if ( v17 == 54 || v17 == 42 )
    {
      byte_140003000 = 1;
    }
    else if ( byte_140003000 == 1 )
    {
      swprintf_s(Dst, 0x100ui64, L"%c", byte_140002270[v17 + 128]);
      v4 = -1i64;
      do
        ++v4;
      while ( Dst[v4] );
      DisplayString.Length = 2 * v4;
      DisplayString.MaximumLength = 2 * v4 + 2;
      DisplayString.Buffer = Dst;
      NtDisplayString(&DisplayString);
      v5 = v17;
      if ( v17 == 14 )
      {
        --v2;
        byte_140003000 = 0;
        NtDelayExecution(0, &Interval);
      }
      else
      {
        if ( v17 <= 0x37u && _bittest64(&v3, v17) || (unsigned __int16)(v17 - 71) <= 0xCu )
        {
          v6 = v2++;
          v7 = v6;
          Dst[v7 + 512] = (unsigned __int8)byte_140003000;
          Dst[v7 + 256] = v5;
        }
        byte_140003000 = 0;
        NtDelayExecution(0, &Interval);
      }
    }
    else
    {
      swprintf_s(Dst, 0x100ui64, L"%c", byte_140002270[v17]);
      v8 = -1i64;
      do
        ++v8;
      while ( Dst[v8] );
      v22.Length = 2 * v8;
      v22.MaximumLength = 2 * v8 + 2;
      v22.Buffer = Dst;
      NtDisplayString(&v22);
      if ( v17 == 14 )
      {
        --v2;
        NtDelayExecution(0, &Interval);
      }
      else if ( v17 <= 0x37u && _bittest64(&v3, v17) || (unsigned __int16)(v17 - 71) <= 0xCu )
      {
        v9 = v2++;
        v10 = v9;
        LOWORD(v9) = (unsigned __int8)byte_140003000;
        Dst[v10 + 256] = v17;
        Dst[v10 + 512] = v9;
        NtDelayExecution(0, &Interval);
      }
      else
      {
LABEL_36:
        NtDelayExecution(0, &Interval);
      }
    }
  }
  Dst[v2 + 256] = 0;
  if ( v2 < 0x27u )
  {
LABEL_35:
    *(_DWORD *)&v14.Length = 5505106;
    v14.Buffer = L"\n\n!!! Incorrect !!!\n\nEnter the password:\n";
    NtDisplayString(&v14);
    v2 = 0;
    goto LABEL_36;
  }
  for ( i = 0; i < v2; ++i )
  {
    if ( Dst[i + 256] != word_140002220[i] || Dst[i + 512] != word_140002370[i] )
      goto LABEL_35;
  }
  *(_DWORD *)&v14.Length = 2621478;
  v14.Buffer = L"\n\n!!! Correct !!!\n\n";
  NtDisplayString(&v14);
  *(_DWORD *)&v14.Length = 4194366;
  v14.Buffer = L"Press any keys to continue...\n\n";
  NtDisplayString(&v14);
  do
  {
    NtReadFile(FileHandle, EventHandle, 0i64, 0i64, &IoStatusBlock, Buffer, 0xCu, &ByteOffset, 0i64);
    NtWaitForSingleObject(EventHandle, 1u, 0i64);
  }
  while ( v18 );
  v12 = 20i64;
  do
  {
    *(_DWORD *)&v14.Length = 262146;
    v14.Buffer = L".";
    NtDisplayString(&v14);
    --v12;
  }
  while ( v12 );
  NtTerminateProcess((HANDLE)0xFFFFFFFFFFFFFFFFi64, 0);
  return 0;
}
```
The logic to print flag is very simple:
+ word_140002220[i] is the scan code of the key the driver expects
+ word_140002370[i] is whether Shift is pressed for that key (1 = Shift, 0 = no Shift)
+ byte_140002270 is basically a keymap from scan code to character (normal and the “+128” part is used when Shift is active)

I wrote a python code to solve this:
```
scan_codes = [
    0x2E, 0x1F, 0x2E, 0x2F, 0x03, 0x0B, 0x03, 0x06, 0x1A, 0x11,
    0x23, 0x05, 0x14, 0x0C, 0x02, 0x2C, 0x0C, 0x31, 0x05, 0x14,
    0x17, 0x2F, 0x04, 0x0C, 0x05, 0x19, 0x19, 0x26, 0x02, 0x2E,
    0x05, 0x14, 0x02, 0x0B, 0x31, 0x02, 0x02, 0x02, 0x1B,
]

shift_flags = [
    1, 1, 1, 1,
    0, 0, 0, 0,
    1, 0,
    0, 0, 0,
    1, 0, 0,
    1, 0,
    0, 0, 0, 0, 0,
    1,
    0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
    1, 1, 1, 1,
]

keymap = [0] * 256

def set_pair(sc, unch, shch):
    keymap[sc]      = ord(unch)
    keymap[sc + 128] = ord(shch)

pairs = [
    (0x02, '1', '!'),
    (0x03, '2', '@'),
    (0x04, '3', '#'),
    (0x05, '4', '$'),
    (0x06, '5', '%'),
    (0x0B, '0', ')'),
    (0x0C, '-', '_'),
    (0x11, 'w', 'W'),
    (0x14, 't', 'T'),
    (0x17, 'i', 'I'),
    (0x19, 'p', 'P'),
    (0x1A, '[', '{'),
    (0x1B, ']', '}'),
    (0x1F, 's', 'S'),
    (0x23, 'h', 'H'),
    (0x26, 'l', 'L'),
    (0x2C, 'z', 'Z'),
    (0x2E, 'c', 'C'),
    (0x2F, 'v', 'V'),
    (0x31, 'n', 'N'),
]

for sc, unch, shch in pairs:
    set_pair(sc, unch, shch)

chars = []
for sc, sf in zip(scan_codes, shift_flags):
    idx = sc + (128 if sf else 0)
    chars.append(chr(keymap[idx]))

password = "".join(chars)
print(password)

```
Output:
```
CSCV2025{wh4t_1z_n4tiv3_4ppl1c4t10n!!!}
```
If program is excercutable, it will response:
```
!!! Correct !!!
Press any keys to continue...
```
=> Flag is: **CSCV2025{wh4t_1z_n4tiv3_4ppl1c4t10n!!!}**

That all !  What a successful competion when my team take place 12th overall 🥰. Thanks for watching this post and see you again in the next post!