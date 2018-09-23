---
layout: post
title: 'Pwn your pal using Github'
---

### Bringing back an old Github "feature" (bug) that can easily set any Github user up

## Intro
It all started when I was browsing sources of one of the apps in our organization on Github. I must say I love browsing code on Github - syntax highlight, line comments, swift loading, a lot of goodness, etc. However, one file listing left me profoundly distracted and confused... The code was mine according to GH but I knew for sure it looks nothing like my code. Just that small patch left me pondering for a while as I was sure it just can't be **my** code, so I started digging it all up to the surface.
<!--more-->
First of all, I didn't see any suspicious activity on my profile page. Nothing off in the Git history as well. The commit bore my name and email. It was a while ago - back when we were not signing our commits (nowadays we certainly do!) within the organization.

## Old "feature"
Then I realized that Github will surely allow any email and name go through once set up via `git config user.name` and `git config user.email`. I quickly checked that by writing Linus' ([@torvalds](https://github.com/torvalds)) name and email which you can get from the git history of the kernel repo. After I pushed the commit, surely enough, I saw Linus "contributing" to our code. What a bummer. However, I did see in my profile page that it was actually me committing to the repo, despite the fact that `@torvalds` username is in the logs and on various pages across Github UI:

![](http://i.imgur.com/cMahgZQ.png, "Oh yeah and John Resig too. Cheers.")

Also see [various graphs](https://github.com/0xBADCA7/sandbox/graphs/contributors). Essentially, it won't matter if you hide your email or not, if I can guess your GH email, it will pick it up upon pushing upstream. However, I could see that it was a fake commit in my profile, just as anybody else in our organization. One side note: the history is valid for one (doh!) month only. In other words if I never spotted it within a month - it's in.

Running forward, I have to say this was my colleague's prank to add a funny patch as I found out later (relying on my SE skills). That wouldn't be so funny if it was _#gotofail_ or alike (_#heartbleed_), right? Google says this was a old "feature" as seen [here](http://www.jayhuang.org/blog/pushing-code-to-github-as-linus-torvalds/) (apparently, Linus' name came up first not only in my mind) and [here](https://news.ycombinator.com/item?id=6918343) (related the the post). There's an email from GH folks presented which describes this as an abusable "feature" on Github. I decided to give it a second hit and send me report-like on the issue. I was more interested in what the GH staff's going to say when I task them with my situation (impersonation). The reply was similar to the one in the post above:

> Hi Megacat,
>
> Thanks for the submission! We have reviewed your report and confirmed
> that this is a known concern. After internally assessing the findings
> we have determined they are low in risk and not eligible for a reward
> under the Bug Bounty program.
>
> It is important to note that this is not a bug, but is a feature of
> GitHub that can be abused. Git is a distributed version control
> system, so it is expected that a repository may contain commits from
> users/emails other than the person pushing those commits to GitHub.
> Impersonating another GitHub user in this fashion doesn't grant you
> access to any of their repositories or give you any privileges you
> didn't already have. However, if someone is wrongfully impersonating
> you, please let us know and we will remove the impersonated commits
> and deal with them as quickly as we can.
>
> Rather than make this feature less useful for everyone who uses it
> responsibly, we strive to make GitHub a fun and safe environment by
> swiftly dealing with bullies and giving you ways to ignore them.
>
> If you are still concerned about this, your team can choose to use
> Git's built in options to sign with a GPG key. If you are concerned
> about having a verifiable identity on your commits, you should check
> into the `git commit -S` command.

And I had all the reasons to be concerned - we did not sign commits back then. I have asked then how to defend myself in case of a two-year old commit that went wrong and what to do if, say, I miss the commit record by another user on my/their profile page and it's lost after one month passed:

> Hi,   You can track all recent activity on GitHub on the contributions
> tab of your profile page. However, this will only show one month's
> worth of information and won't show you all contributions attributed
> to you across all repositories for all of time. If you begin signing
> your commits users would be able to verify the true identity of
> commits attributed to you going forward.
>
> I hope that helps, Patrick

That did not help, of course. Essentially, even after a year or more, there's still no way to know if you were set up or not in the past. Some say this is part of Git that you can supply arbitrary email and name. I don't agree with this statement relatively to GH though. Clearly, GH folks should simply check if committer's/pusher's email is associated with the repository first. So this is a *wontfix*. Now go and wreak _gotofail_ patch havoc till they fix it!

**P.S.** When testing this I've noticed that if the user you're trying to commit in the name of is in the same organization you're in and pushing the commit to, that user's history page will be affected too (in the case above, Linus' history won't show any signs of mischief). I guess this was patched as I can't replicate it anymore.

> Read the manual if unsure, post comment(s) if unclear.