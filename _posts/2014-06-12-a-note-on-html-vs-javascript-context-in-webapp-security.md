---
layout: post
title: 'A note on HTML vs JavaScript context in WebApp security'
---

There's an XSS game/excercise that Google put out there so you could fiddle with XSS [here](https://xss-game.appspot.com/). I was browsing Security.stackexchange.com when stumbled upon a good question: ["How is this XSS attack working?"](https://security.stackexchange.com/questions/60662/how-is-this-xss-attack-working). The question relates to [Level 4](https://xss-game.appspot.com/level4) of the game.

Essentially, after supplying a payload in the form of `https://xss-game.appspot.com/level4/frame/?timer=999')||alert('1`, the source code of the frame would be something like this below:

{% highlight xml %}
...
  <br>
    <img src="/static/loading.gif" onload="startTimer('999&#39;)||alert(&#39;1');" />
    <br>
    <div id="message">Your timer will execute in 999&#39;)||alert(&#39;1 seconds.</div>
...
{% endhighlight %}

As you can witness, single quotes are HTML-encoded which is the right thing to do. However, it is only true in HTML context (hence, _HTML_ encoding). This XSS is being triggered b/c in JavaScript context it is still a valid code, which is a real WTF for many (first, for the developers). The payload did enter the sources of the page by means of POST request to the server-side script which then encoded and embedded it in to JavaScript piece within that page, as opposed to JavaScript processing while page is already _loaded_ (i.e. JavaScript context). This is crazy from the security perspective but that's how it works. To remedy this, developers should _both_ escape (JS context) and encode (HTML context) external data.

> Read the manual if unsure, post comment(s) if unclear.