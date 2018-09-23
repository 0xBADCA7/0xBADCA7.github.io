---
layout: page
title: About/Keys
---

## Trivia
Public records of Netcat here. I write on infosec issues, review CTF tasks, share tools, academic papers and articles that interest me, disclose vulnerabilities and dissect software into bits. This is a web log of @0xBADCA7. Views are my own (as well as models and controllers).

## Blog
This web log is a collection of [memoranda](https://youtu.be/m_2m1qdqieE) made possible thanks to [Jekyll](http://jekyllrb.com/) - a static site compiler. All of the content your browser receives is accessible to everyone on the Web. This site is hosted at [Github](https://github.com) and you can browse its sources [here]({{ site.repo }}).

## Security
The beauty of an open-source site is that anyone can copy ("clone") one to their machine and thus it becomes decentralized. The question of authority then is solved by cryptographically signing each and every difference in the source code ("commit") of this site.

## Keys and verification
The site maintainer's public key(s) can be found [here]({{ site.repo }}/tree/master/public/keys). You should verify the key against some other sources like Twitter or other people, or against only well-known keyservers (the last resort). After having imported the key from the repo, a keyserver or directly me, make sure the fingerprint matches:

~~~
$ gpg --list-keys --with-fingerprint 94173E5CB0D001A0
pub   4096R/0x9EF4A285A3C0BB4E 2015-03-05
      Key fingerprint = CD28 0146 92F1 0D3B C013  01A9 9EF4 A285 A3C0 BB4E
uid                 [ultimate] Megacat (Mawoo) <0xBADCA7@gmail.com>
uid                 [ultimate] Megacat (Blog signing key) <0xBADCA7@gmail.com>
sub   4096R/0xC9AB675C3146EEF4 2015-03-05
sub   4096R/0x94173E5CB0D001A0 2015-03-05
sub   4096R/0x243F9B38C04BCC3C 2015-05-10

gpg> check
uid  Megacat (Mawoo) <0xBADCA7@gmail.com>
sig!3        0x9EF4A285A3C0BB4E 2015-05-10  [self-signature]
uid  Megacat (Blog signing key) <0xBADCA7@gmail.com>
sig!3        0x9EF4A285A3C0BB4E 2015-05-10  [self-signature]

~~~

Each commit (either separately or under a tag) is signed with my blog signing key `0x243F9B38C04BCC3C` or `0x94173E5CB0D001A0`. You can check the authenticity of posts after pulling down the code, changing to its directory and issuing `git tag` command in order to list tags and then `git tag -v <tag>`; or `git log --show-signature` to see signatures per commit if a signed tag is missing. See more information on how to sign and verify with Git [here](http://git-scm.com/book/en/v2/Git-Tools-Signing-Your-Work). When verifying a commit, output like this can be observed:

~~~
$ git log --show-signature
commit c705c10018b416707a70279dd2100f58982a7893
gpg: Signature made Sun May 10 23:10:24 2015 CEST
gpg:                using RSA key 0x243F9B38C04BCC3C
gpg: Good signature from "Megacat (Mawoo) <0xBADCA7@gmail.com>" [ultimate]
gpg:                 aka "Megacat (Blog signing key) <0xBADCA7@gmail.com>" [ultimate]
Author: 0xBADCA7 <0xBADCA7@gmail.com>
Date:   Sun May 10 23:10:24 2015 +0200

    Scaffold up Jekyll and migrate old posts
(END)
~~~

<p class="message">
  If you suspect something is not matching or there's an issue - create an issue over at Github and email me at the address associated with the keys. Much appreciated.
</p>

