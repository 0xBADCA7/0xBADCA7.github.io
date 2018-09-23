---
layout: post
title: SU-CTF 2016 web250 "Old Persian"
tags: CTF, SU-CTF, Sharif
---

The challenge was to log in as user "admin" to a web page with a log-in form that bears a custom CAPTCHA. Legend:

>**Old persian cuneiform captcha**
Old Persian cuneiform is a semi-alphabetic cuneiform script that was the primary script for the old persian language. You could get more information on following links, 
- http://www.ancientscripts.com/oldpersian.html 
- https://en.wikipedia.org/wiki/Old_Persian_cuneiform.
>
>A web-based collections management for a museum has some extremely valuable information if one has admin user access. [The Site ](javascript:document.write('Each team had a personalized site URL')). We found that the "admin" user have a 4-digit password. But they use a captcha made of 10 old persian characters. One has to use the correspondence between symbols and strings to pass theye captcha verification (use "trans.png"). Log in as "admin" to find the flag. 
the flag is in the fomat: \[Your flag is: flagflagflag...\] (without braces)
[Download](http://ctf.sharif.edu/2016/panel/challenges/131/download/)
 <!--more-->

The translation image (the "Download" link) is this one below:

![](http://i.imgur.com/uMa7Urm.png)

The page itself looks like that:

![](http://i.imgur.com/9D0cnzL.png)

We know from the task description that the user we're attacking is "admin" and the password ranges from 0000 to 9999 which is 10^4 combinations – quite doable over the Internet. Alas, the captcha needs to be either solved or bypassed.

Aiming for the low-hanging fruit, I decided first to take a look at the verification mechanism in order to figure out whether we really need to solve captchas or not. After submitting the log-in form the server-side script would send the 302 header and redirect back to the log-in page which would then render either "Invalid captcha" or "Login failed" depending on correctness of the solution. The curious thing I found was that it was possible to issue an arbitrary number of POST
requests with credentials set and (obviously) wrong captcha within the same connection (thanks to `Connection: Keep-Alive` header) then visit the login page again to see multiple "Invalid captcha" messages. Looked like a winner to me – we could get one captcha solved correctly by hand and then try all the passwords, scooping up the results from one page. Alas, only the first attempt was a valid one, the rest needed correct captchas. Enter *Plan B*. 

Clearly,  I needed to solve captchas either with a hack or Machine Learning. I figured out that the byte size is unique to every captcha and perhaps, it was possible to infer a solution based on image byte size but then we wouldn't be able to distinguish between any two images with the same set of symbols having their positions swapped. That amounts to quite a lot of combinations if considering manual correction. I equally noticed that the symbols are only slightly ( ≤ 45°) rotated
the either way. This means I needed only 3-5 distinct variations per symbol to successfully identify them using ML. I could have gone the Python way and use libraries like PIL, CV, etc. however this time I decided to go with *Wolfram Mathematica* 10 for much more fun.

Mathematica 10 provides a nice, handy high-level access to ML. The learning set would be something like 7-10 minutes of work to put together which I swiftly did. Below is the script I used to classify symbols in the captcha with error rate circa 1/100 at worst. Sources are [here](https://gist.github.com/0xBADCA7/50fc3cf21a5c2232eee3).

![](http://i.imgur.com/9oTTFPv.gif)

Using the `Classify` function I was able to painlessly recognize text behind symbols. Code is well commented so no need for further explanation except for the note that there's an accompanying Python `proxy.py` script that facilitates fetching of fresh captchas and submitting the log-in form. It could have been done directly in Mathematica however it was not able to maintain session and every time a new captcha was fetched and solved – it would already be invalid for the challenge's
server script upon submission. Thus, I used Python requests module to create and maintain the session. There are traces of Mathematica's HTTP headers in the proxy script which is the result of me trying to make the two programs look like the same client, however something was wrong with the set up (most likely Mathematica needed `Connection: Keep-Alive`  header as well) and thus I deemed it quicker to put up a Python script better.


> Read the manual if unsure, post comment(s) if unclear.
