---
layout: post
title: Reverse engineering of Viscosity VPN client
---

This is a tutorial on how to teach Viscosity 1.4.7 (1222) to stay "registered" at any time so one can use it without any restrictions whatsoever even after their trial period has expired. This is merely a tutorial and is written for educational purposes only. Although I don't really imply any certain use of this document, I still suggest buying a license of the software that you like and use every day. Perhaps, it's healthy to do the both - purchase a license _and_ reverse engineer that s/w altogether. <!--more-->

### Tools needed
* Sparklabs Viscosity version 1.4.7 (1222)
* Windows 8.1 machine (others might do as well - never tried)
* ILDASM (ildasm.exe) from the Windows SDK ([this one](http://msdn.microsoft.com/en-us/windows/desktop/bg162891.aspx) in my case for Windows 8). You then may find it in `C:\Program Files (x86)\Microsoft SDKs\Windows\v8.1A\bin\NETFX 4.5.1 Tools` or a similar location.
* CFF Explorer VIII from [here](www.ntcore.com/exsuite.php). Helps browsing and patching .Net assemblies.
* [Telerik JustDecompile](http://www.telerik.com/products/decompiler.aspx) - a .Net decompiler to C# or other high level .Net languages, to have a higher level view of the code we'll work on.
* De4Dot plugin for Telerik JustDecompile which can be found [here](https://bitbucket.org/0xd4d/de4dot/) or [there](https://github.com/0xd4d/de4dot), as we'll need to deobfuscate the code first.

### Deobfuscate
If you try to decompile the binary in its original form, you'll encounter a bunch of nonsensical symbols. This is a typical trace of a code obfuscator. Having successfully installed *JustDecompile* and its *De4Dot* plugin, open up the `Viscosity.exe` with it, select the root of the assembly tree, choose `De4dot > Obfuscator search` from the selection's secondary menu. The plugin will successfully identify a packer/obfuscator and offer us to save the "cleaned up" version. Feel free to place it anywhere on your disk and then load it up again with the same JustDecompile. Leave the window open as sometimes you might want to take a look at the references across the code.

### Disassemble
Open `ildasm.exe` and drag-n-drop the very same cleaned up executable from the previous step. From `File` choose `Dump` and tick the checkboxes as in this [snapshot](http://i.imgur.com/mZcfkUv.png). This will help us see real bytes when patching the binary in a hex editor. From now on, if you double-click any method you will be presented with a window with the method's disassembly.

![Tick boxes like this](http://i.imgur.com/mZcfkUv.png)

### Analyze behavior
When Viscosity starts, it presents you a form with a request to register or purchase (when the trial period is over). This means we need to find the code responsible for verification or proof of registration, but even better - the one that initiates it. Go to *JustDecompile* and pick *GoTo Entry Point* from the secondary menu when selecting the root of assembly:

![Go to entry point](http://i.imgur.com/yfgYiGs.png)

If you look at the code for a while you'll notice that one of the final statements is `CMainForm cMainForm = new CMainForm();`. This is logical but we made sure no tomfoolery is going on before the main form has been shown. Now, if you look at the list of forms to the left (in *JustDecompile*) you'll notice another `CfrmRegister` form, exploring which quickly ensures us that the form is being presented when the application requires us to register or buy a license when newly opened. 

Right-click `CfrmRegister` and choose *Find Usage* from the secondary menu. A window just popped up will show all the references to `CfrmRegister`. Among one of the scopes is the `CMainForm`. Clearly, we need to find when and why the registration form is being called from within the `CMainForm` methods. 

Staring a little more at the methods, we may find that there's that `method_59` one which does all the evil. The method is declared as `void` so I suggest we can simply return back from function right after entering it.

### Patch
Pull the ILDASM's window once again, find the `method_59` reference under `CMainForm` and disassemble it by double-clicking the former. You should see the listing below:

~~~ildasm
.method private hidebysig instance void  method_59() cil managed
// SIG: 20 00 01
{
  // Method begins at RVA 0x20758
  // Code size       1120 (0x460)
  .maxstack  11
  .locals init (valuetype [mscorlib]System.DateTime V_0,
           int64 V_1,
           int64 V_2,
           int64 V_3,
           int64 V_4,
           string V_5,
           int32 V_6,
           int32 V_7,
           valuetype [mscorlib]System.DateTime V_8,
           class [mscorlib]Microsoft.Win32.RegistryKey V_9,
           class [mscorlib]System.Collections.Generic.List`1<int64> V_10,
           int64 V_11,
           int32 V_12,
           object V_13,
           class CfrmRegister V_14,
           class [mscorlib]System.Collections.Generic.List`1<int64> V_15)
  IL_0000:  /* 28   | (0A)00013F       */ call       valuetype [mscorlib]System.DateTime [mscorlib]System.DateTime::get_Now()
  IL_0005:  /* 0A   |                  */ stloc.0
  IL_0006:  /* 28   | (0A)00013F       */ call       valuetype [mscorlib]System.DateTime [mscorlib]System.DateTime::get_Now()
  IL_000b:  /* 28   | (06)000443       */ call       int64 GClass32::smethod_12(valuetype [mscorlib]System.DateTime)
  IL_0010:  /* 0B   |                  */ stloc.1
  IL_0011:  /* 28   | (06)000446       */ call       string GClass32::smethod_15()
  IL_0016:  /* 13   | 05               */ stloc.s    V_5
  IL_0018:  /* 11   | 05               */ ldloc.s    V_5
  IL_001a:  /* 28   | (0A)000072       */ call       bool [mscorlib]System.IO.File::Exists(string)
  IL_001f:  /* 3A   | 8F000000         */ brtrue     IL_00b3
  IL_0024:  /* 11   | 05               */ ldloc.s    V_5
  IL_0026:  /* 02   |                  */ ldarg.0
  IL_0027:  /* 12   | 01               */ ldloca.s   V_1
  IL_0029:  /* 28   | (0A)00017C       */ call       instance string [mscorlib]System.Int64::ToString()
  IL_002e:  /* 28   | (06)0001D8       */ call       instance string CMainForm::method_49(string)
  IL_0033:  /* 28   | (0A)000178       */ call       class [mscorlib]System.Text.Encoding [mscorlib]System.Text.Encoding::get_UTF8()
  IL_0038:  /* 28   | (0A)000290       */ call       void [mscorlib]System.IO.File::WriteAllText(string,
                                                                                                 string,
                                                                                                 class [mscorlib]System.Text.Encoding)
  IL_003d:  /* 73   | (0A)000291       */ newobj     instance void [mscorlib]System.Random::.ctor()
  IL_0042:  /* 17   |                  */ ldc.i4.1
  IL_0043:  /* 12   | 00               */ ldloca.s   V_0
  IL_0045:  /* 28   | (0A)000292       */ call       instance int32 [mscorlib]System.DateTime::get_Day()
  IL_004a:  /* 6F   | (0A)000293       */ callvirt   instance int32 [mscorlib]System.Random::Next(int32,
                                                                                                  int32)
  IL_004f:  /* 13   | 06               */ stloc.s    V_6
  IL_0051:  /* 73   | (0A)000291       */ newobj     instance void [mscorlib]System.Random::.ctor()
  IL_0056:  /* 17   |                  */ ldc.i4.1
  IL_0057:  /* 12   | 00               */ ldloca.s   V_0
  IL_0059:  /* 28   | (0A)000294       */ call       instance int32 [mscorlib]System.DateTime::get_Month()
  IL_005e:  /* 6F   | (0A)000293       */ callvirt   instance int32 [mscorlib]System.Random::Next(int32,
                                                                                                  int32)
  IL_0063:  /* 13   | 07               */ stloc.s    V_7
  IL_0065:  /* 12   | 08               */ ldloca.s   V_8
  IL_0067:  /* 12   | 00               */ ldloca.s   V_0
  IL_0069:  /* 28   | (0A)000295       */ call       instance int32 [mscorlib]System.DateTime::get_Year()
  IL_006e:  /* 11   | 07               */ ldloc.s    V_7
  IL_0070:  /* 11   | 06               */ ldloc.s    V_6
  IL_0072:  /* 12   | 00               */ ldloca.s   V_0
  IL_0074:  /* 28   | (0A)000296       */ call       instance int32 [mscorlib]System.DateTime::get_Hour()
  IL_0079:  /* 12   | 00               */ ldloca.s   V_0
  IL_007b:  /* 28   | (0A)000297       */ call       instance int32 [mscorlib]System.DateTime::get_Minute()
  IL_0080:  /* 12   | 00               */ ldloca.s   V_0
  IL_0082:  /* 28   | (0A)000298       */ call       instance int32 [mscorlib]System.DateTime::get_Second()
  IL_0087:  /* 28   | (0A)000299       */ call       instance void [mscorlib]System.DateTime::.ctor(int32,
                                                                                                    int32,
                                                                                                    int32,
                                                                                                    int32,
                                                                                                    int32,
                                                                                                    int32)
  IL_008c:  /* 11   | 05               */ ldloc.s    V_5
  IL_008e:  /* 11   | 08               */ ldloc.s    V_8
  IL_0090:  /* 28   | (0A)00029A       */ call       void [mscorlib]System.IO.File::SetCreationTime(string,
                                                                                                    valuetype [mscorlib]System.DateTime)
  IL_0095:  /* 11   | 05               */ ldloc.s    V_5
  IL_0097:  /* 11   | 08               */ ldloc.s    V_8
  IL_0099:  /* 28   | (0A)00029B       */ call       void [mscorlib]System.IO.File::SetLastAccessTime(string,
                                                                                                      valuetype [mscorlib]System.DateTime)
  IL_009e:  /* 11   | 05               */ ldloc.s    V_5
  IL_00a0:  /* 11   | 08               */ ldloc.s    V_8
  IL_00a2:  /* 28   | (0A)00029C       */ call       void [mscorlib]System.IO.File::SetLastWriteTime(string,
                                                                                                     valuetype [mscorlib]System.DateTime)
  IL_00a7:  /* 11   | 05               */ ldloc.s    V_5
  IL_00a9:  /* 18   |                  */ ldc.i4.2
  IL_00aa:  /* 28   | (0A)0000A9       */ call       void [mscorlib]System.IO.File::SetAttributes(string,
                                                                                                  valuetype [mscorlib]System.IO.FileAttributes)
  IL_00af:  /* 07   |                  */ ldloc.1
  IL_00b0:  /* 0C   |                  */ stloc.2
  IL_00b1:  /* 2B   | 20               */ br.s       IL_00d3
  .try
  {
    IL_00b3:  /* 02   |                  */ ldarg.0
    IL_00b4:  /* 11   | 05               */ ldloc.s    V_5
    IL_00b6:  /* 28   | (0A)000178       */ call       class [mscorlib]System.Text.Encoding [mscorlib]System.Text.Encoding::get_UTF8()
    IL_00bb:  /* 28   | (0A)00029D       */ call       string [mscorlib]System.IO.File::ReadAllText(string,
                                                                                                    class [mscorlib]System.Text.Encoding)
    IL_00c0:  /* 28   | (06)0001D9       */ call       instance string CMainForm::method_50(string)
    IL_00c5:  /* 28   | (0A)00029E       */ call       float64 [mscorlib]System.Convert::ToDouble(string)
    IL_00ca:  /* 6A   |                  */ conv.i8
    IL_00cb:  /* 0C   |                  */ stloc.2
    IL_00cc:  /* DE   | 05               */ leave.s    IL_00d3
  }  // end .try
  catch [mscorlib]System.Object 
  {
    IL_00ce:  /* 26   |                  */ pop
    IL_00cf:  /* 07   |                  */ ldloc.1
    IL_00d0:  /* 0C   |                  */ stloc.2
    IL_00d1:  /* DE   | 00               */ leave.s    IL_00d3
  }  // end handler
  // HEX: 00 00 B3 00 1B CE 00 05 01 00 00 01
  IL_00d3:  /* 7E   | (0A)00029F       */ ldsfld     class [mscorlib]Microsoft.Win32.RegistryKey [mscorlib]Microsoft.Win32.Registry::CurrentUser
  IL_00d8:  /* 72   | (70)006F0A       */ ldstr      "SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Explorer"
  IL_00dd:  /* 17   |                  */ ldc.i4.1
  IL_00de:  /* 6F   | (0A)0002A0       */ callvirt   instance class [mscorlib]Microsoft.Win32.RegistryKey [mscorlib]Microsoft.Win32.RegistryKey::OpenSubKey(string,
                                                                                                                                                            bool)
  IL_00e3:  /* 13   | 09               */ stloc.s    V_9
  IL_00e5:  /* 11   | 09               */ ldloc.s    V_9
  IL_00e7:  /* 72   | (70)006F70       */ ldstr      "FileFoundBandHook"
  IL_00ec:  /* 6F   | (0A)0002A1       */ callvirt   instance object [mscorlib]Microsoft.Win32.RegistryKey::GetValue(string)
  IL_00f1:  /* 2D   | 1E               */ brtrue.s   IL_0111
  IL_00f3:  /* 11   | 09               */ ldloc.s    V_9
  IL_00f5:  /* 72   | (70)006F70       */ ldstr      "FileFoundBandHook"
  IL_00fa:  /* 02   |                  */ ldarg.0
  IL_00fb:  /* 12   | 01               */ ldloca.s   V_1
  IL_00fd:  /* 28   | (0A)00017C       */ call       instance string [mscorlib]System.Int64::ToString()
  IL_0102:  /* 28   | (06)0001DA       */ call       instance string CMainForm::method_51(string)
  IL_0107:  /* 6F   | (0A)0002A2       */ callvirt   instance void [mscorlib]Microsoft.Win32.RegistryKey::SetValue(string,
                                                                                                                   object)
  IL_010c:  /* 07   |                  */ ldloc.1
  IL_010d:  /* 13   | 04               */ stloc.s    V_4
  IL_010f:  /* 2B   | 27               */ br.s       IL_0138
  .try
  {
    IL_0111:  /* 02   |                  */ ldarg.0
    IL_0112:  /* 11   | 09               */ ldloc.s    V_9
    IL_0114:  /* 72   | (70)006F70       */ ldstr      "FileFoundBandHook"
    IL_0119:  /* 6F   | (0A)0002A1       */ callvirt   instance object [mscorlib]Microsoft.Win32.RegistryKey::GetValue(string)
    IL_011e:  /* 74   | (01)000003       */ castclass  [mscorlib]System.String
    IL_0123:  /* 28   | (06)0001D9       */ call       instance string CMainForm::method_50(string)
    IL_0128:  /* 28   | (0A)00029E       */ call       float64 [mscorlib]System.Convert::ToDouble(string)
    IL_012d:  /* 6A   |                  */ conv.i8
    IL_012e:  /* 13   | 04               */ stloc.s    V_4
    IL_0130:  /* DE   | 06               */ leave.s    IL_0138
  }  // end .try
  catch [mscorlib]System.Object 
  {
    IL_0132:  /* 26   |                  */ pop
    IL_0133:  /* 07   |                  */ ldloc.1
    IL_0134:  /* 13   | 04               */ stloc.s    V_4
    IL_0136:  /* DE   | 00               */ leave.s    IL_0138
  }  // end handler
  // HEX: 00 00 11 01 21 32 01 06 01 00 00 01
  IL_0138:  /* 72   | (70)006F94       */ ldstr      "Launch"
  IL_013d:  /* 28   | (06)000351       */ call       string GClass22::smethod_8(string)
  IL_0142:  /* 7E   | (0A)00000A       */ ldsfld     string [mscorlib]System.String::Empty
  IL_0147:  /* 28   | (0A)000034       */ call       bool [mscorlib]System.String::op_Equality(string,
                                                                                               string)
  IL_014c:  /* 2C   | 1B               */ brfalse.s  IL_0169
  IL_014e:  /* 72   | (70)006F94       */ ldstr      "Launch"
  IL_0153:  /* 02   |                  */ ldarg.0
  IL_0154:  /* 12   | 01               */ ldloca.s   V_1
  IL_0156:  /* 28   | (0A)00017C       */ call       instance string [mscorlib]System.Int64::ToString()
  IL_015b:  /* 28   | (06)0001D8       */ call       instance string CMainForm::method_49(string)
  IL_0160:  /* 28   | (06)000352       */ call       void GClass22::smethod_9(string,
                                                                              string)
  IL_0165:  /* 07   |                  */ ldloc.1
  IL_0166:  /* 0D   |                  */ stloc.3
  IL_0167:  /* 2B   | 1E               */ br.s       IL_0187
  .try
  {
    IL_0169:  /* 02   |                  */ ldarg.0
    IL_016a:  /* 72   | (70)006F94       */ ldstr      "Launch"
    IL_016f:  /* 28   | (06)000351       */ call       string GClass22::smethod_8(string)
    IL_0174:  /* 28   | (06)0001D9       */ call       instance string CMainForm::method_50(string)
    IL_0179:  /* 28   | (0A)00029E       */ call       float64 [mscorlib]System.Convert::ToDouble(string)
    IL_017e:  /* 6A   |                  */ conv.i8
    IL_017f:  /* 0D   |                  */ stloc.3
    IL_0180:  /* DE   | 05               */ leave.s    IL_0187
  }  // end .try
  catch [mscorlib]System.Object 
  {
    IL_0182:  /* 26   |                  */ pop
    IL_0183:  /* 07   |                  */ ldloc.1
    IL_0184:  /* 0D   |                  */ stloc.3
    IL_0185:  /* DE   | 00               */ leave.s    IL_0187
  }  // end handler
  // HEX: 00 00 69 01 19 82 01 05 01 00 00 01
  IL_0187:  /* 73   | (0A)00018E       */ newobj     instance void class [mscorlib]System.Collections.Generic.List`1<int64>::.ctor()
  IL_018c:  /* 13   | 0F               */ stloc.s    V_15
  IL_018e:  /* 11   | 0F               */ ldloc.s    V_15
  IL_0190:  /* 08   |                  */ ldloc.2
  IL_0191:  /* 6F   | (0A)000190       */ callvirt   instance void class [mscorlib]System.Collections.Generic.List`1<int64>::Add(!0)
  IL_0196:  /* 11   | 0F               */ ldloc.s    V_15
  IL_0198:  /* 09   |                  */ ldloc.3
  IL_0199:  /* 6F   | (0A)000190       */ callvirt   instance void class [mscorlib]System.Collections.Generic.List`1<int64>::Add(!0)
  IL_019e:  /* 11   | 0F               */ ldloc.s    V_15
  IL_01a0:  /* 11   | 04               */ ldloc.s    V_4
  IL_01a2:  /* 6F   | (0A)000190       */ callvirt   instance void class [mscorlib]System.Collections.Generic.List`1<int64>::Add(!0)
  IL_01a7:  /* 11   | 0F               */ ldloc.s    V_15
  IL_01a9:  /* 13   | 0A               */ stloc.s    V_10
  IL_01ab:  /* 11   | 0A               */ ldloc.s    V_10
  IL_01ad:  /* 6F   | (0A)0002A3       */ callvirt   instance void class [mscorlib]System.Collections.Generic.List`1<int64>::Sort()
  IL_01b2:  /* 11   | 0A               */ ldloc.s    V_10
  IL_01b4:  /* 16   |                  */ ldc.i4.0
  IL_01b5:  /* 6F   | (0A)0002A4       */ callvirt   instance !0 class [mscorlib]System.Collections.Generic.List`1<int64>::get_Item(int32)
  IL_01ba:  /* 13   | 0B               */ stloc.s    V_11
  IL_01bc:  /* 08   |                  */ ldloc.2
  IL_01bd:  /* 11   | 0B               */ ldloc.s    V_11
  IL_01bf:  /* 31   | 2D               */ ble.s      IL_01ee
  .try
  {
    IL_01c1:  /* 11   | 05               */ ldloc.s    V_5
    IL_01c3:  /* 20   | 80000000         */ ldc.i4     0x80
    IL_01c8:  /* 28   | (0A)0000A9       */ call       void [mscorlib]System.IO.File::SetAttributes(string,
                                                                                                    valuetype [mscorlib]System.IO.FileAttributes)
    IL_01cd:  /* 11   | 05               */ ldloc.s    V_5
    IL_01cf:  /* 02   |                  */ ldarg.0
    IL_01d0:  /* 12   | 0B               */ ldloca.s   V_11
    IL_01d2:  /* 28   | (0A)00017C       */ call       instance string [mscorlib]System.Int64::ToString()
    IL_01d7:  /* 28   | (06)0001D8       */ call       instance string CMainForm::method_49(string)
    IL_01dc:  /* 28   | (0A)0002A5       */ call       void [mscorlib]System.IO.File::WriteAllText(string,
                                                                                                   string)
    IL_01e1:  /* 11   | 05               */ ldloc.s    V_5
    IL_01e3:  /* 18   |                  */ ldc.i4.2
    IL_01e4:  /* 28   | (0A)0000A9       */ call       void [mscorlib]System.IO.File::SetAttributes(string,
                                                                                                    valuetype [mscorlib]System.IO.FileAttributes)
    IL_01e9:  /* DE   | 03               */ leave.s    IL_01ee
  }  // end .try
  catch [mscorlib]System.Object 
  {
    IL_01eb:  /* 26   |                  */ pop
    IL_01ec:  /* DE   | 00               */ leave.s    IL_01ee
  }  // end handler
  // HEX: 00 00 C1 01 2A EB 01 03 01 00 00 01
  IL_01ee:  /* 09   |                  */ ldloc.3
  IL_01ef:  /* 11   | 0B               */ ldloc.s    V_11
  IL_01f1:  /* 31   | 17               */ ble.s      IL_020a
  IL_01f3:  /* 72   | (70)006F94       */ ldstr      "Launch"
  IL_01f8:  /* 02   |                  */ ldarg.0
  IL_01f9:  /* 12   | 0B               */ ldloca.s   V_11
  IL_01fb:  /* 28   | (0A)00017C       */ call       instance string [mscorlib]System.Int64::ToString()
  IL_0200:  /* 28   | (06)0001D8       */ call       instance string CMainForm::method_49(string)
  IL_0205:  /* 28   | (06)000352       */ call       void GClass22::smethod_9(string,
                                                                              string)
  IL_020a:  /* 11   | 04               */ ldloc.s    V_4
  IL_020c:  /* 11   | 0B               */ ldloc.s    V_11
  IL_020e:  /* 31   | 1E               */ ble.s      IL_022e
  .try
  {
    IL_0210:  /* 11   | 09               */ ldloc.s    V_9
    IL_0212:  /* 72   | (70)006F70       */ ldstr      "FileFoundBandHook"
    IL_0217:  /* 02   |                  */ ldarg.0
    IL_0218:  /* 12   | 0B               */ ldloca.s   V_11
    IL_021a:  /* 28   | (0A)00017C       */ call       instance string [mscorlib]System.Int64::ToString()
    IL_021f:  /* 28   | (06)0001DA       */ call       instance string CMainForm::method_51(string)
    IL_0224:  /* 6F   | (0A)0002A2       */ callvirt   instance void [mscorlib]Microsoft.Win32.RegistryKey::SetValue(string,
                                                                                                                     object)
    IL_0229:  /* DE   | 03               */ leave.s    IL_022e
  }  // end .try
  catch [mscorlib]System.Object 
  {
    IL_022b:  /* 26   |                  */ pop
    IL_022c:  /* DE   | 00               */ leave.s    IL_022e
  }  // end handler
  // HEX: 00 00 10 02 1B 2B 02 03 01 00 00 01
  IL_022e:  /* 16   |                  */ ldc.i4.0
  IL_022f:  /* 13   | 0C               */ stloc.s    V_12
  IL_0231:  /* 07   |                  */ ldloc.1
  IL_0232:  /* 11   | 0B               */ ldloc.s    V_11
  IL_0234:  /* 2F   | 05               */ bge.s      IL_023b
  IL_0236:  /* 16   |                  */ ldc.i4.0
  IL_0237:  /* 13   | 0C               */ stloc.s    V_12
  IL_0239:  /* 2B   | 1E               */ br.s       IL_0259
  .try
  {
    IL_023b:  /* 1F   | 1E               */ ldc.i4.s   30
    IL_023d:  /* 6A   |                  */ conv.i8
    IL_023e:  /* 07   |                  */ ldloc.1
    IL_023f:  /* 11   | 0B               */ ldloc.s    V_11
    IL_0241:  /* 59   |                  */ sub
    IL_0242:  /* 20   | 80510100         */ ldc.i4     0x15180
    IL_0247:  /* 6A   |                  */ conv.i8
    IL_0248:  /* 5B   |                  */ div
    IL_0249:  /* 59   |                  */ sub
    IL_024a:  /* 28   | (0A)0002A6       */ call       int32 [mscorlib]System.Convert::ToInt32(int64)
    IL_024f:  /* 13   | 0C               */ stloc.s    V_12
    IL_0251:  /* DE   | 06               */ leave.s    IL_0259
  }  // end .try
  catch [mscorlib]System.Object 
  {
    IL_0253:  /* 26   |                  */ pop
    IL_0254:  /* 16   |                  */ ldc.i4.0
    IL_0255:  /* 13   | 0C               */ stloc.s    V_12
    IL_0257:  /* DE   | 00               */ leave.s    IL_0259
  }  // end handler
  // HEX: 00 00 3B 02 18 53 02 06 01 00 00 01
  IL_0259:  /* 14   |                  */ ldnull
  IL_025a:  /* 13   | 0D               */ stloc.s    V_13
  IL_025c:  /* 11   | 0C               */ ldloc.s    V_12
  IL_025e:  /* 16   |                  */ ldc.i4.0
  IL_025f:  /* 30   | 3B               */ bgt.s      IL_029c
  IL_0261:  /* 02   |                  */ ldarg.0
  IL_0262:  /* 28   | (06)000189       */ call       instance void CMainForm::method_9()
  IL_0267:  /* 72   | (70)006FA2       */ ldstr      "TRIALEXPIRED"
  IL_026c:  /* 72   | (70)006FBC       */ ldstr      "Trial Period Expired!"
  IL_0271:  /* 28   | (06)00022D       */ call       string GClass10::smethod_2(string,
                                                                                string)
  IL_0276:  /* 72   | (70)006FE8       */ ldstr      "TRIALEXPIREDDESC"
  IL_027b:  /* 72   | (70)00700A       */ ldstr      "Your trial period has expired. If you like Viscosi"
  + "ty please purchase it for unrestricted use."
  IL_0280:  /* 28   | (06)00022D       */ call       string GClass10::smethod_2(string,
                                                                                string)
  IL_0285:  /* 16   |                  */ ldc.i4.0
  IL_0286:  /* 14   |                  */ ldnull
  IL_0287:  /* 14   |                  */ ldnull
  IL_0288:  /* 17   |                  */ ldc.i4.1
  IL_0289:  /* 16   |                  */ ldc.i4.0
  IL_028a:  /* 16   |                  */ ldc.i4.0
  IL_028b:  /* 16   |                  */ ldc.i4.0
  IL_028c:  /* 17   |                  */ ldc.i4.1
  IL_028d:  /* 16   |                  */ ldc.i4.0
  IL_028e:  /* 28   | (06)00043C       */ call       object[] GClass32::smethod_5(string,
                                                                                  string,
                                                                                  bool,
                                                                                  object,
                                                                                  object,
                                                                                  bool,
                                                                                  bool,
                                                                                  bool,
                                                                                  bool,
                                                                                  bool,
                                                                                  bool)
  IL_0293:  /* 16   |                  */ ldc.i4.0
  IL_0294:  /* 9A   |                  */ ldelem.ref
  IL_0295:  /* 13   | 0D               */ stloc.s    V_13
  IL_0297:  /* 38   | F0000000         */ br         IL_038c
  IL_029c:  /* 11   | 0C               */ ldloc.s    V_12
  IL_029e:  /* 17   |                  */ ldc.i4.1
  IL_029f:  /* 30   | 35               */ bgt.s      IL_02d6
  IL_02a1:  /* 72   | (70)0070C7       */ ldstr      "TRAILWARNING"
  IL_02a6:  /* 72   | (70)0070E1       */ ldstr      "Please purchase Viscosity!"
  IL_02ab:  /* 28   | (06)00022D       */ call       string GClass10::smethod_2(string,
                                                                                string)
  IL_02b0:  /* 72   | (70)007117       */ ldstr      "TRAILWARNING1DAYDESC"
  IL_02b5:  /* 72   | (70)007141       */ ldstr      "Your trial period expires in 1 day! If you like Vi"
  + "scosity please purchase it for unrestricted use."
  IL_02ba:  /* 28   | (06)00022D       */ call       string GClass10::smethod_2(string,
                                                                                string)
  IL_02bf:  /* 16   |                  */ ldc.i4.0
  IL_02c0:  /* 14   |                  */ ldnull
  IL_02c1:  /* 14   |                  */ ldnull
  IL_02c2:  /* 17   |                  */ ldc.i4.1
  IL_02c3:  /* 16   |                  */ ldc.i4.0
  IL_02c4:  /* 16   |                  */ ldc.i4.0
  IL_02c5:  /* 16   |                  */ ldc.i4.0
  IL_02c6:  /* 16   |                  */ ldc.i4.0
  IL_02c7:  /* 17   |                  */ ldc.i4.1
  IL_02c8:  /* 28   | (06)00043C       */ call       object[] GClass32::smethod_5(string,
                                                                                  string,
                                                                                  bool,
                                                                                  object,
                                                                                  object,
                                                                                  bool,
                                                                                  bool,
                                                                                  bool,
                                                                                  bool,
                                                                                  bool,
                                                                                  bool)
  IL_02cd:  /* 16   |                  */ ldc.i4.0
  IL_02ce:  /* 9A   |                  */ ldelem.ref
  IL_02cf:  /* 13   | 0D               */ stloc.s    V_13
  IL_02d1:  /* 38   | B6000000         */ br         IL_038c
  IL_02d6:  /* 11   | 0C               */ ldloc.s    V_12
  IL_02d8:  /* 1B   |                  */ ldc.i4.5
  IL_02d9:  /* 30   | 58               */ bgt.s      IL_0333
  IL_02db:  /* 72   | (70)007208       */ ldstr      "5DayWarning"
  IL_02e0:  /* 28   | (06)00034F       */ call       bool GClass22::smethod_6(string)
  IL_02e5:  /* 3A   | A2000000         */ brtrue     IL_038c
  IL_02ea:  /* 72   | (70)0070C7       */ ldstr      "TRAILWARNING"
  IL_02ef:  /* 72   | (70)0070E1       */ ldstr      "Please purchase Viscosity!"
  IL_02f4:  /* 28   | (06)00022D       */ call       string GClass10::smethod_2(string,
                                                                                string)
  IL_02f9:  /* 72   | (70)007220       */ ldstr      "TRAILWARNINGDESC"
  IL_02fe:  /* 72   | (70)007242       */ ldstr      "Your trial period expires in {0:d} days. If you li"
  + "ke Viscosity please purchase it for unrestricted use."
  IL_0303:  /* 28   | (06)00022D       */ call       string GClass10::smethod_2(string,
                                                                                string)
  IL_0308:  /* 11   | 0C               */ ldloc.s    V_12
  IL_030a:  /* 8C   | (01)00003A       */ box        [mscorlib]System.Int32
  IL_030f:  /* 28   | (0A)00005F       */ call       string [mscorlib]System.String::Format(string,
                                                                                            object)
  IL_0314:  /* 16   |                  */ ldc.i4.0
  IL_0315:  /* 14   |                  */ ldnull
  IL_0316:  /* 14   |                  */ ldnull
  IL_0317:  /* 17   |                  */ ldc.i4.1
  IL_0318:  /* 16   |                  */ ldc.i4.0
  IL_0319:  /* 16   |                  */ ldc.i4.0
  IL_031a:  /* 16   |                  */ ldc.i4.0
  IL_031b:  /* 16   |                  */ ldc.i4.0
  IL_031c:  /* 17   |                  */ ldc.i4.1
  IL_031d:  /* 28   | (06)00043C       */ call       object[] GClass32::smethod_5(string,
                                                                                  string,
                                                                                  bool,
                                                                                  object,
                                                                                  object,
                                                                                  bool,
                                                                                  bool,
                                                                                  bool,
                                                                                  bool,
                                                                                  bool,
                                                                                  bool)
  IL_0322:  /* 16   |                  */ ldc.i4.0
  IL_0323:  /* 9A   |                  */ ldelem.ref
  IL_0324:  /* 13   | 0D               */ stloc.s    V_13
  IL_0326:  /* 72   | (70)007208       */ ldstr      "5DayWarning"
  IL_032b:  /* 17   |                  */ ldc.i4.1
  IL_032c:  /* 28   | (06)000350       */ call       void GClass22::smethod_7(string,
                                                                              bool)
  IL_0331:  /* 2B   | 59               */ br.s       IL_038c
  IL_0333:  /* 11   | 0C               */ ldloc.s    V_12
  IL_0335:  /* 1F   | 0A               */ ldc.i4.s   10
  IL_0337:  /* 30   | 53               */ bgt.s      IL_038c
  IL_0339:  /* 72   | (70)007313       */ ldstr      "10DayWarning"
  IL_033e:  /* 28   | (06)00034F       */ call       bool GClass22::smethod_6(string)
  IL_0343:  /* 2D   | 47               */ brtrue.s   IL_038c
  IL_0345:  /* 72   | (70)0070C7       */ ldstr      "TRAILWARNING"
  IL_034a:  /* 72   | (70)0070E1       */ ldstr      "Please purchase Viscosity!"
  IL_034f:  /* 28   | (06)00022D       */ call       string GClass10::smethod_2(string,
                                                                                string)
  IL_0354:  /* 72   | (70)007220       */ ldstr      "TRAILWARNINGDESC"
  IL_0359:  /* 72   | (70)007242       */ ldstr      "Your trial period expires in {0:d} days. If you li"
  + "ke Viscosity please purchase it for unrestricted use."
  IL_035e:  /* 28   | (06)00022D       */ call       string GClass10::smethod_2(string,
                                                                                string)
  IL_0363:  /* 11   | 0C               */ ldloc.s    V_12
  IL_0365:  /* 8C   | (01)00003A       */ box        [mscorlib]System.Int32
  IL_036a:  /* 28   | (0A)00005F       */ call       string [mscorlib]System.String::Format(string,
                                                                                            object)
  IL_036f:  /* 16   |                  */ ldc.i4.0
  IL_0370:  /* 14   |                  */ ldnull
  IL_0371:  /* 14   |                  */ ldnull
  IL_0372:  /* 17   |                  */ ldc.i4.1
  IL_0373:  /* 16   |                  */ ldc.i4.0
  IL_0374:  /* 16   |                  */ ldc.i4.0
  IL_0375:  /* 16   |                  */ ldc.i4.0
  IL_0376:  /* 16   |                  */ ldc.i4.0
  IL_0377:  /* 17   |                  */ ldc.i4.1
  IL_0378:  /* 28   | (06)00043C       */ call       object[] GClass32::smethod_5(string,
                                                                                  string,
                                                                                  bool,
                                                                                  object,
                                                                                  object,
                                                                                  bool,
                                                                                  bool,
                                                                                  bool,
                                                                                  bool,
                                                                                  bool,
                                                                                  bool)
  IL_037d:  /* 16   |                  */ ldc.i4.0
  IL_037e:  /* 9A   |                  */ ldelem.ref
  IL_037f:  /* 13   | 0D               */ stloc.s    V_13
  IL_0381:  /* 72   | (70)007313       */ ldstr      "10DayWarning"
  IL_0386:  /* 17   |                  */ ldc.i4.1
  IL_0387:  /* 28   | (06)000350       */ call       void GClass22::smethod_7(string,
                                                                              bool)
  IL_038c:  /* 11   | 0D               */ ldloc.s    V_13
  IL_038e:  /* 39   | C1000000         */ brfalse    IL_0454
  IL_0393:  /* 11   | 0D               */ ldloc.s    V_13
  IL_0395:  /* 6F   | (0A)0002A7       */ callvirt   instance class [mscorlib]System.Type [mscorlib]System.Object::GetType()
  IL_039a:  /* D0   | (01)0000E6       */ ldtoken    [mscorlib]System.Boolean
  IL_039f:  /* 28   | (0A)0000AB       */ call       class [mscorlib]System.Type [mscorlib]System.Type::GetTypeFromHandle(valuetype [mscorlib]System.RuntimeTypeHandle)
  IL_03a4:  /* 28   | (0A)0002A8       */ call       bool [mscorlib]System.Type::op_Equality(class [mscorlib]System.Type,
                                                                                             class [mscorlib]System.Type)
  IL_03a9:  /* 2C   | 19               */ brfalse.s  IL_03c4
  IL_03ab:  /* 11   | 0D               */ ldloc.s    V_13
  IL_03ad:  /* A5   | (01)0000E6       */ unbox.any  [mscorlib]System.Boolean
  IL_03b2:  /* 2D   | 10               */ brtrue.s   IL_03c4
  IL_03b4:  /* 72   | (70)0050C5       */ ldstr      "https://secure.sparklabs.com/store/viscosity"
  IL_03b9:  /* 28   | (0A)0001C5       */ call       class [System]System.Diagnostics.Process [System]System.Diagnostics.Process::Start(string)
  IL_03be:  /* 26   |                  */ pop
  IL_03bf:  /* 38   | 90000000         */ br         IL_0454
  IL_03c4:  /* 11   | 0D               */ ldloc.s    V_13
  IL_03c6:  /* 6F   | (0A)0002A7       */ callvirt   instance class [mscorlib]System.Type [mscorlib]System.Object::GetType()
  IL_03cb:  /* D0   | (01)000003       */ ldtoken    [mscorlib]System.String
  IL_03d0:  /* 28   | (0A)0000AB       */ call       class [mscorlib]System.Type [mscorlib]System.Type::GetTypeFromHandle(valuetype [mscorlib]System.RuntimeTypeHandle)
  IL_03d5:  /* 28   | (0A)0002A8       */ call       bool [mscorlib]System.Type::op_Equality(class [mscorlib]System.Type,
                                                                                             class [mscorlib]System.Type)
  IL_03da:  /* 2C   | 78               */ brfalse.s  IL_0454
  IL_03dc:  /* 11   | 0D               */ ldloc.s    V_13
  IL_03de:  /* 74   | (01)000003       */ castclass  [mscorlib]System.String
  IL_03e3:  /* 72   | (70)002E35       */ ldstr      "Append"
  IL_03e8:  /* 28   | (0A)000034       */ call       bool [mscorlib]System.String::op_Equality(string,
                                                                                               string)
  IL_03ed:  /* 2C   | 65               */ brfalse.s  IL_0454
  IL_03ef:  /* 73   | (06)0000E6       */ newobj     instance void CfrmRegister::.ctor()
  IL_03f4:  /* 13   | 0E               */ stloc.s    V_14
  IL_03f6:  /* 11   | 0E               */ ldloc.s    V_14
  IL_03f8:  /* 17   |                  */ ldc.i4.1
  IL_03f9:  /* 6F   | (0A)00009A       */ callvirt   instance void [System.Windows.Forms]System.Windows.Forms.Form::set_StartPosition(valuetype [System.Windows.Forms]System.Windows.Forms.FormStartPosition)
  IL_03fe:  /* 11   | 0E               */ ldloc.s    V_14
  IL_0400:  /* 6F   | (0A)00009B       */ callvirt   instance valuetype [System.Windows.Forms]System.Windows.Forms.DialogResult [System.Windows.Forms]System.Windows.Forms.Form::ShowDialog()
  IL_0405:  /* 17   |                  */ ldc.i4.1
  IL_0406:  /* 33   | 46               */ bne.un.s   IL_044e
  IL_0408:  /* 02   |                  */ ldarg.0
  IL_0409:  /* 28   | (06)000188       */ call       instance void CMainForm::method_8()
  IL_040e:  /* 02   |                  */ ldarg.0
  IL_040f:  /* 11   | 0E               */ ldloc.s    V_14
  IL_0411:  /* 7B   | (04)00012B       */ ldfld      class [System.Windows.Forms]System.Windows.Forms.TextBox CfrmRegister::tbName
  IL_0416:  /* 6F   | (0A)000035       */ callvirt   instance string [System.Windows.Forms]System.Windows.Forms.Control::get_Text()
  IL_041b:  /* 28   | (06)0001EF       */ call       instance void CMainForm::method_56(string)
  IL_0420:  /* 72   | (70)006DA9       */ ldstr      "REGOSUCCESS"
  IL_0425:  /* 72   | (70)006DC1       */ ldstr      "Registration Successful"
  IL_042a:  /* 28   | (06)00022D       */ call       string GClass10::smethod_2(string,
                                                                                string)
  IL_042f:  /* 72   | (70)006DF1       */ ldstr      "REGOSUCCESSDESC"
  IL_0434:  /* 72   | (70)006E11       */ ldstr      "Registration was successful. All limits have been "
  + "removed. Thank you for purchasing Viscosity!"
  IL_0439:  /* 28   | (06)00022D       */ call       string GClass10::smethod_2(string,
                                                                                string)
  IL_043e:  /* 16   |                  */ ldc.i4.0
  IL_043f:  /* 14   |                  */ ldnull
  IL_0440:  /* 14   |                  */ ldnull
  IL_0441:  /* 17   |                  */ ldc.i4.1
  IL_0442:  /* 16   |                  */ ldc.i4.0
  IL_0443:  /* 16   |                  */ ldc.i4.0
  IL_0444:  /* 16   |                  */ ldc.i4.0
  IL_0445:  /* 16   |                  */ ldc.i4.0
  IL_0446:  /* 16   |                  */ ldc.i4.0
  IL_0447:  /* 28   | (06)00043C       */ call       object[] GClass32::smethod_5(string,
                                                                                  string,
                                                                                  bool,
                                                                                  object,
                                                                                  object,
                                                                                  bool,
                                                                                  bool,
                                                                                  bool,
                                                                                  bool,
                                                                                  bool,
                                                                                  bool)
  IL_044c:  /* 26   |                  */ pop
  IL_044d:  /* 2A   |                  */ ret
  IL_044e:  /* 15   |                  */ ldc.i4.m1
  IL_044f:  /* 28   | (0A)000212       */ call       void [mscorlib]System.Environment::Exit(int32)
  IL_0454:  /* 11   | 0C               */ ldloc.s    V_12
  IL_0456:  /* 16   |                  */ ldc.i4.0
  IL_0457:  /* 30   | 06               */ bgt.s      IL_045f
  IL_0459:  /* 15   |                  */ ldc.i4.m1
  IL_045a:  /* 28   | (0A)000212       */ call       void [mscorlib]System.Environment::Exit(int32)
  IL_045f:  /* 2A   |                  */ ret
} // end of method CMainForm::method_59

~~~

Very well. ILDASM reports that the method's *relative offset* (RVA) is at `0x20758` (`// Method begins at RVA 0x20758` at the top of disassembly). Now, switch to *CFF Explorer* window (presumably you opened the same binary there already) and select *Address Converter* from the left hand side. In the `RVA` field at the top of the window type in our `0x20758` to see the real file offset below. 

![](http://i.imgur.com/HhWmpo3.png)

Once you have the real file offset, go to *Hex Editor* on the left hand side and then jump to it. Find the bytes that ILDASM is reporting as the first code of the method (`28   | (0A)00013F`). You should see a sequence of bytes `28 3f 01 00 0A` and so on. The method `method_59` is declared as `void` so I suggest we can simply return right after entering it, as I mentioned above. For that all you have to do is replace byte `28` with `2A` which is an instruction to return from a call. You can do it either right there, in *CFF Explorer*, or in your favorite hex editor. Save the binary, back up and replace the original, and you'll have a Viscosity client that never asks to register anything.


#### Coda
This post has demonstrated binary patching of .Net applications and suggested some nice tools to work your way through reverse engineering of .Net applications. We, essentially, have replaced a single instruction in the binary. However, a good patch/crack would need more QA, so to speak, in order to see if there are conditions which cause exceptions. Patching instructions like we did is a pretty poor but an effective choice. The reason is that what we did is not really the same as writing code and placing a `return` statement as the first one in the method closure (it's easy to see that, if you code your own simple program and then disassemble it). Compilers/interpreters _manage_ the source code and thus there might be several areas you actually need to complement your original byte replacement with. I'll do my best to make a post on how to do it correctly as well.


----------

Some additional links on binary patching .Net apps:

* [http://fr33kk0mpu73r.blogspot.com.tr/2013/10/cracking-net-applications-diffenginex.html](http://fr33kk0mpu73r.blogspot.com.tr/2013/10/cracking-net-applications-diffenginex.html)
* [http://resources.infosecinstitute.com/patching-net-binary-code-with-cff-explorer/](http://resources.infosecinstitute.com/patching-net-binary-code-with-cff-explorer/)


----------


> Read the manual if unsure, post comment(s) if unclear.

