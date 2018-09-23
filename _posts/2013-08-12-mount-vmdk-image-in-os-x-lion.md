---
layout: post
title: Mount VMDK image in OS X (Lion)
---

I was working on this lesson [here][1] (Memory Corruption p.2) and it looks like there was no `vulnerable.ocx` ActiveX control attached to the lecture post. As there were some VM images [available][2] for students attending [HackNight][3] events, I thought that they might contain the sought .ocx.

I downloaded the image file(s) and then when I tried to launch one of those with Windows XP I received a lovely offer to activate my XP copy. Looked like I had to work with the hard drive anyway (either find and extract the ActiveX control if existed or suppress activation). I decided to simply attach the image to my OS X Lion (10.7.5). Here are the steps to repeat it:

  * Fetch Q (Qemu front-end) for Mac at [http://www.kju-app.org/][4]

  * After installing Q crack open terminal and convert the `.vmdk` to a raw image first like this (fixing the paths to fit your system best, of course): `$ /Applications/Q.app/Contents/MacOS/qemu-img convert -f vmdk ~/Downloads/WindowsXPPro.vmdk ~/Downloads/xppro.raw`

  * Next, find out where the actual data starts inside the image by invoking:
        `$ fdisk ~/Downloads/xppro.raw`

The output should be similar to the one below:

<pre class="code bash">
Disk: /Users/0xBADCA7/Downloads/xppro.raw	geometry: 2610/255/63 [41943040 sectors]
Signature: 0xAA55
         Starting       Ending
 #: id  cyl  hd sec -  cyl  hd sec [     start -       size]
------------------------------------------------------------------------
*1: 07    0   1   1 - 1023 254  63 [        63 -   41913522] HPFS/QNX/AUX
 2: 00    0   0   0 -    0   0   0 [         0 -          0] unused
 3: 00    0   0   0 -    0   0   0 [         0 -          0] unused
 4: 00    0   0   0 -    0   0   0 [         0 -          0] unused
</pre>
<br>
You can tell that the offset we need is "63".

  * Create a folder where you'd want the image to be mounted at
 with `$ mkdir /tmp/xppro`

  * Attach the disk image  `$ hdid -section 63 -nomount -imagekey diskimage-class=CRawDiskImage ~/Downloads/xppro.raw`

  The expected output is the path to the newly created disk like `/dev/disk1`

  * Mount the disk to the directory created two steps back

    <code>$ sudo mount_ntfs /dev/disk1 /tmp/xppro</code>

  * **Done**. List VM's hard drive contents now:

<pre class="code bash">
$ ls /tmp/xppro

total 3146320
-rwxr-xr-x@  1 0xbadca7  masters   244K Apr 14  2008 ntldr
-rwxr-xr-x@  1 0xbadca7  masters    46K Apr 14  2008 NTDETECT.COM
-rwxr-xr-x@  1 0xbadca7  masters   211B Apr 24 04:57 boot.ini
prwxr-xr-x   1 0xbadca7  masters     0B Apr 24 05:35 MSDOS.SYS
prwxr-xr-x   1 0xbadca7  masters     0B Apr 24 05:35 IO.SYS
-rwxr-xr-x   1 0xbadca7  masters     0B Apr 24 05:35 CONFIG.SYS
-rwxr-xr-x   1 0xbadca7  masters     0B Apr 24 05:35 AUTOEXEC.BAT
drwxr-xr-x@  1 0xbadca7  masters   4.0K Apr 24 05:59 System Volume Information
drwxr-xr-x   1 0xbadca7  masters   4.0K Apr 24 06:00 Documents and Settings
drwxr-xr-x   1 0xbadca7  masters    28K Apr 24 06:00 WINDOWS
drwxr-xr-x   1 0xbadca7  masters   4.0K Apr 24 06:00 Program Files
-rwxr-xr-x@  1 0xbadca7  masters   1.5G Aug  6 12:21 pagefile.sys
drwxr-xr-x   1 0xbadca7  masters   4.0K Aug  6 12:36 .
drwxrwxrwt  29 root   wheelee   986B Aug  6 13:04 ..
</pre>
<br>
Now you can look for anything on the VM's hard drive. _Hint_: I used `ack` tool to save some time typing out a bunch of `find`s and `grep`s every time. Can be found [here][5]. If you managed to do the same in OS X _Mountain Lion_ please let me know, I'll update the post.

Read the manual if unsure, post comment(s) if unclear.

P.S.: Never found that `vulnerable.ocx`. Will code it up myself.

[1]: http://pentest.cryptocity.net/exploitation/exploitation-102.html "Memory Corruption part 2 with Alex Sotirov"
[2]: http://isis.poly.edu/~hake/hacknight/ "VM instances you'd need"
[3]: http://www.isis.poly.edu/hack-night "Hack Night @ ISIS NY Poly"
[4]: http://www.kju-app.org/ "Qemu front-end called Kju-app now"
[5]: http://beyondgrep.com/ "The ack tool's homepage"
