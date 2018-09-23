---
layout: post
title: Chrome Sniff-Snuff extension
---

I wrote a simple but effective Chrome Developer Tools extension that lets one sniff canaries – specific crafted values one can distinguish from the rest of information easily – among headers, content and cookies of web pages under assessment.

![Chrome Sniff-Snuff extension](https://raw.githubusercontent.com/0xBADCA7/chrome-sniff-snuff/master/sniffsnuff.png)

 <!--more-->There's plenty of software and extensions to do MitM on web traffic but then that would require one to fire up Burp/ZAP/other proxy when with Sniff-Snuff you don't really need to do that much. It's not a MitM tool anyway. Also, you might not have such tools handy on Chromebooks or office computers.

I needed something to quickly see where my XSS payloads may appear after injection. I would craft a canary like `<script>alert('unique_value_31337')</script>` (depending on the case of course) and then see if there are any AJAX requests being fired that may contain the payload (which would mean it goes somewhere else and that node can be vulnerable & exploitable) or forms being submitted away from the application elsewhere, headers or cookies affected, etc.

Usage is quite simple: type in Javascript style regex into the search field, check options underneath and hit the search button. If there's a need to modify the scope – stop the sniffer, de/activate some options and hit the search button again.

See the extension on Github [here](https://github.com/0xBADCA7/chrome-sniff-snuff).
