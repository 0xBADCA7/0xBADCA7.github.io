---
layout: post
title: Network proxy in OS X from command line
---

I was dealing with quick switching between different proxies in OS X Mountain Lion and decided to automate that process at some point via Terminal (any command line). I have written a tiny Bash routine that would set a new HTTP proxy and activate it and deactivate one. See the Gist below. I will add more routines to deal with HTTPS, SOCKS4/5, etc.

This might come in handy when working with proxies from command line, in case you need a system-wide proxy setting


```
# Proxy-related routines for OSX

function proxy_on() {
    #sudo
    networksetup -setwebproxy "$3" $1 $2 && sudo networksetup -setwebproxystate "$3" on
}

function proxy_off() {
    #sudo
    networksetup -setwebproxystate "$1" off
}

```

