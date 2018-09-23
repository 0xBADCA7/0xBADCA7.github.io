---
layout: post
title: NoConName Facebook CTF 2013 quals write-up
---

### Intro
[This](https://ctftime.org/event/109 "Description @ ctftime.org") CTF was really short, fun and intended for beginners (my guess). Three tasks only, weird [rules](http://noconname.org/files/CTF_NocONName_2013_ENG.pdf "That's a PDF") and, allegedly, sponsored by *Facebook*.

![](http://i.imgur.com/GzoPdw7.png)

### Level 1
You are faced with a web page having a web form. The form bears an input field for the _key_. A JavaScript method is bound to the `submit` event of the form:

    <form action="login.php" method="POST" onsubmit="return encrypt(this);">


A `crypto.js` JavaScript file is attached to the page where `encrypt` et al are located in an obfuscated form:


    var _0x52ae=["\x66\x20\x6F\x28\x38\x29\x7B\x63\x20\x69\x2C\x6A\x3D\x30\x3B\x6B\x28\x69\x3D\x30\x3B\x69\x3C\x38\x2E\x6C\x3B\x69\x2B\x2B\x29\x7B\x6A\x2B\x3D\x28\x38\x5B\x69\x5D\x2E\x73\x28\x29\x2A\x28\x69\x2B\x31\x29\x29\x7D\x67\x20\x74\x2E\x75\x28\x6A\x29\x25\x76\x7D\x66\x20\x70\x28\x68\x29\x7B\x68\x3D\x68\x2E\x71\x28\x30\x29\x3B\x63\x20\x69\x3B\x6B\x28\x69\x3D\x30\x3B\x69\x3C\x77\x3B\x2B\x2B\x69\x29\x7B\x63\x20\x35\x3D\x69\x2E\x78\x28\x79\x29\x3B\x6D\x28\x35\x2E\x6C\x3D\x3D\x31\x29\x35\x3D\x22\x30\x22\x2B\x35\x3B\x35\x3D\x22\x25\x22\x2B\x35\x3B\x35\x3D\x7A\x28\x35\x29\x3B\x6D\x28\x35\x3D\x3D\x68\x29\x41\x7D\x67\x20\x69\x7D\x66\x20\x6E\x28\x38\x29\x7B\x63\x20\x69\x2C\x61\x3D\x30\x2C\x62\x3B\x6B\x28\x69\x3D\x30\x3B\x69\x3C\x38\x2E\x6C\x3B\x2B\x2B\x69\x29\x7B\x62\x3D\x70\x28\x38\x2E\x71\x28\x69\x29\x29\x3B\x61\x2B\x3D\x62\x2A\x28\x69\x2B\x31\x29\x7D\x67\x20\x61\x7D\x66\x20\x42\x28\x39\x29\x7B\x63\x20\x32\x3B\x32\x3D\x6E\x28\x39\x2E\x64\x2E\x65\x29\x3B\x32\x3D\x32\x2A\x28\x33\x2B\x31\x2B\x33\x2B\x33\x2B\x37\x29\x3B\x32\x3D\x32\x3E\x3E\x3E\x36\x3B\x32\x3D\x32\x2F\x34\x3B\x32\x3D\x32\x5E\x43\x3B\x6D\x28\x32\x21\x3D\x30\x29\x7B\x72\x28\x27\x44\x20\x64\x21\x27\x29\x7D\x45\x7B\x72\x28\x27\x46\x20\x64\x20\x3A\x29\x27\x29\x7D\x39\x2E\x47\x2E\x65\x3D\x6E\x28\x39\x2E\x64\x2E\x65\x29\x3B\x39\x2E\x48\x2E\x65\x3D\x22\x49\x22\x2B\x6F\x28\x39\x2E\x64\x2E\x65\x29\x3B\x67\x20\x4A\x7D","\x7C","\x73\x70\x6C\x69\x74","\x7C\x7C\x72\x65\x73\x7C\x7C\x7C\x68\x65\x78\x5F\x69\x7C\x7C\x7C\x73\x74\x72\x7C\x66\x6F\x72\x6D\x7C\x7C\x7C\x76\x61\x72\x7C\x70\x61\x73\x73\x77\x6F\x72\x64\x7C\x76\x61\x6C\x75\x65\x7C\x66\x75\x6E\x63\x74\x69\x6F\x6E\x7C\x72\x65\x74\x75\x72\x6E\x7C\x66\x6F\x6F\x7C\x7C\x68\x61\x73\x68\x7C\x66\x6F\x72\x7C\x6C\x65\x6E\x67\x74\x68\x7C\x69\x66\x7C\x6E\x75\x6D\x65\x72\x69\x63\x61\x6C\x5F\x76\x61\x6C\x75\x65\x7C\x73\x69\x6D\x70\x6C\x65\x48\x61\x73\x68\x7C\x61\x73\x63\x69\x69\x5F\x6F\x6E\x65\x7C\x63\x68\x61\x72\x41\x74\x7C\x61\x6C\x65\x72\x74\x7C\x63\x68\x61\x72\x43\x6F\x64\x65\x41\x74\x7C\x4D\x61\x74\x68\x7C\x61\x62\x73\x7C\x33\x31\x33\x33\x37\x7C\x32\x35\x36\x7C\x74\x6F\x53\x74\x72\x69\x6E\x67\x7C\x31\x36\x7C\x75\x6E\x65\x73\x63\x61\x70\x65\x7C\x62\x72\x65\x61\x6B\x7C\x65\x6E\x63\x72\x79\x70\x74\x7C\x34\x31\x35\x33\x7C\x49\x6E\x76\x61\x6C\x69\x64\x7C\x65\x6C\x73\x65\x7C\x43\x6F\x72\x72\x65\x63\x74\x7C\x6B\x65\x79\x7C\x76\x65\x72\x69\x66\x69\x63\x61\x74\x69\x6F\x6E\x7C\x79\x65\x73\x7C\x74\x72\x75\x65","","\x66\x72\x6F\x6D\x43\x68\x61\x72\x43\x6F\x64\x65","\x72\x65\x70\x6C\x61\x63\x65","\x5C\x77\x2B","\x5C\x62","\x67"];eval(function (_0x7038x1,_0x7038x2,_0x7038x3,_0x7038x4,_0x7038x5,_0x7038x6){_0x7038x5=function (_0x7038x3){return (_0x7038x3<_0x7038x2?_0x52ae[4]:_0x7038x5(parseInt(_0x7038x3/_0x7038x2)))+((_0x7038x3=_0x7038x3%_0x7038x2)>35?String[_0x52ae[5]](_0x7038x3+29):_0x7038x3.toString(36));} ;if(!_0x52ae[4][_0x52ae[6]](/^/,String)){while(_0x7038x3--){_0x7038x6[_0x7038x5(_0x7038x3)]=_0x7038x4[_0x7038x3]||_0x7038x5(_0x7038x3);} ;_0x7038x4=[function (_0x7038x5){return _0x7038x6[_0x7038x5];} ];_0x7038x5=function (){return _0x52ae[7];} ;_0x7038x3=1;} ;while(_0x7038x3--){if(_0x7038x4[_0x7038x3]){_0x7038x1=_0x7038x1[_0x52ae[6]]( new RegExp(_0x52ae[8]+_0x7038x5(_0x7038x3)+_0x52ae[8],_0x52ae[9]),_0x7038x4[_0x7038x3]);} ;} ;return _0x7038x1;} (_0x52ae[0],46,46,_0x52ae[3][_0x52ae[2]](_0x52ae[1]),0,{}));


That's no biggie for us, thanks to Chrome's _Dev Tools_ which deobfuscate such nastiness:

    > encrypt
    > function encrypt(form){var res;res=numerical_value(form.password.value);res=res*(3+1+3+3+7);res=res>>>6;res=res/4;res=res^4153;if(res!=0){alert('Invalid password!')}else{alert('Correct password :)')}form.key.value=numerical_value(form.password.value);form.verification.value="yes"+simpleHash(form.password.value);return true}

After throwing it into [JSBeautifier](http://jsbeautifier.org/):

    function encrypt(form) {
    var res;
    res = numerical_value(form.password.value);
    res = res * (3 + 1 + 3 + 3 + 7);
    res = res >>> 6;
    res = res / 4;
    res = res ^ 4153;
    if (res != 0) {
        alert('Invalid password!')
    } else {
        alert('Correct password :)')
    }
    form.key.value = numerical_value(form.password.value);
    form.verification.value = "yes" + simpleHash(form.password.value);
    return true
}

Now it's plain to see that we need to examine `numerical_value` and, less importantly, `simpleHash` functions.

#### numerical_value()
    function numerical_value(str) {
    var i, a = 0,
        b;
    for (i = 0; i < str.length; ++i) {
        b = ascii_one(str.charAt(i));
        a += b * (i + 1)
    }
    return a
    }

#### simpleHash()

    function simpleHash(str) {
    var i, hash = 0;
    for (i = 0; i < str.length; i++) {
        hash += (str[i].charCodeAt() * (i + 1))
    }
    return Math.abs(hash) % 31337
    }

`numerical_value` is parent to `ascii_one` function which is:

#### ascii_one()

    function ascii_one(foo) {
    foo = foo.charAt(0);
    var i;
    for (i = 0; i < 256; ++i) {
        var hex_i = i.toString(16);
        if (hex_i.length == 1) hex_i = "0" + hex_i;
        hex_i = "%" + hex_i;
        hex_i = unescape(hex_i);
        if (hex_i == foo) break
    }
    return i
    }

`ascii_one()`, essentially, returns a _hex-value_ of its only parameter - a character from ASCII table, so that `ascii_one('A')` is `65` and `ascii_one('\xFF')` is `255`, which is the maximum value that `ascii_one` returns (we will need this later on).

`numerical_value()` returns a value (a number) of the string passed as its only parameter, so that `numerical_value('A')` is also `65`, however `numerical_value('AA')` is `195` (due to the `a += b * (i + 1)` piece).

Getting back to `encrypt()` we can reverse up to the value of `res` that would become `0` after these operations:

    0: res = numerical_value(form.password.value);
    1: res = res * (3 + 1 + 3 + 3 + 7);
    2: res = res >>> 6;
    3: res = res / 4;
    4: res = res ^ 4153;

`form.password.value` is the _key_ input field, by the way. In order to be `0` after line _4_, `res` has to equal `4153` when ⊕ with `4153`. Then 4153 is multiplied by 4 which brings `16612`. Now, the curious bit: `>>>` is the [unsigned right shift](http://msdn.microsoft.com/en-us/library/342xfs5s%28v=vs.94%29.aspx) operator in JavaScript (I guess, the only unsigned operator in JavaScript if there was nothing new recently or I was under a rock).

>The `>>>` operator shifts the bits of expression1 right by the number of bits specified in expression2. Zeroes are filled in from the left. Digits shifted off the right are discarded.

This means that there might be _several_ numbers that would result into `16612` after `>>>` by 6. To find some of them quickly I ran this code in the console:

    a = []; for (var i=0;i<=99999999;i++){ (i >>> 6) == 16612 && a.push(i) }

`a[]` contains a bunch of candidates now. On line _1_, `res` (a candidate from `a[]`) needs to be divided by prime `17`. Note, that `numerical_value()` returns an integer and so only the numbers from `a[]` that are divisible by 17 without a remainder are of interest. Sample `a[]` contents:

    [1063168, 1063169, 1063170, 1063171, 1063172, 1063173, 1063174, 1063175, 1063176, 1063177, 1063178, 1063179, 1063180, 1063181, 1063182, 1063183, 1063184, 1063185, 1063186, 1063187, 1063188, 1063189, 1063190, 1063191, 1063192, 1063193, 1063194, 1063195, 1063196, 1063197, 1063198, 1063199, 1063200, 1063201, 1063202, 1063203, 1063204, 1063205, 1063206, 1063207, 1063208, 1063209, 1063210, 1063211, 1063212, 1063213, 1063214, 1063215, 1063216, 1063217, 1063218, 1063219, 1063220, 1063221, 1063222, 1063223, 1063224, 1063225, 1063226, 1063227, 1063228, 1063229, 1063230, 1063231]

Quickly checking randomly selected items:

    1063168 >>> 6
    16612
    1063210 >>> 6
    16612

Now we need only those that pass the _no-remainder_ requirement:
<pre>
for (i in a) { console.log(a[i]/17) }
62539.294117647056
62539.35294117647
62539.41176470588
62539.470588235294
62539.529411764706
62539.58823529412
62539.64705882353
62539.705882352944
62539.76470588235
62539.82352941176
62539.882352941175
62539.94117647059
62540
62540.05882352941
62540.117647058825
62540.17647058824
62540.23529411765
62540.294117647056
62540.35294117647
62540.41176470588
62540.470588235294
62540.529411764706
62540.58823529412
62540.64705882353
62540.705882352944
62540.76470588235
62540.82352941176
62540.882352941175
62540.94117647059
62541
62541.05882352941
62541.117647058825
62541.17647058824
62541.23529411765
62541.294117647056
62541.35294117647
62541.41176470588
62541.470588235294
62541.529411764706
62541.58823529412
62541.64705882353
62541.705882352944
62541.76470588235
62541.82352941176
62541.882352941175
62541.94117647059
62542
62542.05882352941
62542.117647058825
62542.17647058824
62542.23529411765
62542.294117647056
62542.35294117647
62542.41176470588
62542.470588235294
62542.529411764706
62542.58823529412
62542.64705882353
62542.705882352944
62542.76470588235
62542.82352941176
62542.882352941175
62542.94117647059
62543
</pre>

We have only a handful left now:

    b = []; for (i in a) { !(a[i]%17) && b.push(a[i])  }
    4
    b
    [1063180, 1063197, 1063214, 1063231]


Perfect, now we need `numerical_value()` to return one of `[62540, 62541, 62542, 62543]` and we're done. That's easier with bigger integers in the line ` a = []; for (var i=0;i<=99999999;i++){ (i >>> 6) == 16612 && a.push(i) }` as the `b[]` array <s>would</s>may get even larger and we would have more space to approximate. There's, actually, a mathematical approach to this approximation but, really, it's elementary to do it by hand.

Here are the facts we know about `numerical_value`'s return values:

-   It's an integer no greater than `255` ('\xFF') multiplied by its position:
-   It's an integer that can be as low as `0` ('\x00') irrespectively of its position in the string, that's why `numerical_value('\x00\x00\x00')` is still `0`
-   It's an integer deliberately selected, provided that '\x01' is a character in the desired position, so that `numerical_value('\x00\x00\x01')` becomes `3`

That's enough to get what we want:

    var s = '\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xA4\x00\x00\x00\x00\x01'; // This is only one of many variants


Test:

    > numerical_value(s)
    > 62540

*Gotcha!* Now, the rest is peanuts:

    document.forms[0].password.value = s;
    document.forms[0].key.value = numerical_value(document.forms[0].password.value);
    document.forms[0].verification.value = "yes" + simpleHash(document.forms[0].password.value);
    document.forms[0].submit();

> Invalid password!

Bad news. My browser (sane) won't let null-bytes into form fields:

    > s.length
    27
    > document.forms[0].password.value.length
    22

We have to remove `'\x00'` from `s`:

    var s = '\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFA\xAA';

Trying again:

    Congrats! you passed the level! Here is the key: 23f8d1cea8d60c5816700892284809a94bd00fe7347645b96a99559749c7b7b8


### Level 2

You are given an Android _apk_. I rushed to install it inside my Android emulator:

    adb install level.apk

Alas, the app never started. I don't know if it was supposed to be so or was it a technical issue, but I have disassembled the app with _apktool_ and started looking thru the code. I found multiple references to bitmaps and so navigated to the _/res_ folder where app's resources like images are normally placed. Discovered was a pile of images looking like pieces of a _QR_ code image:

![](http://i.imgur.com/JTBxyqY.png)

I zoomed in and, instead of programmatically recreating the QR code, decided to open up this guy ↓

![](http://i.imgur.com/aJk8SOu.png)


Each image was 97x97px, totally 16 images - that's a 4x4 grid on a (97px x 4) height/width canvas. I nearly momentously recreated the QR code with drag-n-drop as there were clearly four corners already available among the sliced images:

![](http://i.imgur.com/ZkKImAl.png)

I shoved it into [Web QR](http://www.webqr.com/) and obtained the flag:

![](http://i.imgur.com/vdbpEK3.png)


### Level 3
Given a file `level.elf`. Running `file` against it shows it's an ELF64 binary. The program waits for input on the console and quits after a single key hit (except _space_, as I noticed, which outputs an asterisk and then again listens for input - probably `getchar()`). Running `strings` on the binary shows a whole bunch of flag look-alikes, as it normally should.

Disassemled the binary and carefully looked for the reason _space_ has been accepted as the first character. Indeed, at address 0x4010F3 the program checks for the input char to be `0x20` (_space_ key code) and then jumps to the next iteration of `getchar()`.

![](http://i.imgur.com/fqOOr5g.png)

It's easy to spot the `success` procedure down there. All you'd need is to point _IP_ to location 0x40117B and the program will run as nothing ever happened:

![](http://i.imgur.com/kOxHeSR.png)

Run till return and fetch the flag:

![](http://i.imgur.com/5WomToJ.png)


> Read the manual if unsure, post comment(s) if unclear.
