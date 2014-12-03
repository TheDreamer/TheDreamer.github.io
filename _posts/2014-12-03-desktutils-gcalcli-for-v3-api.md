---
title: updated deskutils/gcalcli to use Google's V3 Calendar API
layout: post
comments: yes
category: patches
---

On November 17th, Google pulled the plug on its deprecated V2 Calendar API.  Found that gcalcli had apparently undergone
a complete(?) rewrite to support V3 differently, but no release had been made/tagged.  So, I devised a quick and dirty
patch to grab its state on `20141031`.  So, created a PORTVERSION of either 3.1.a.20141031 or 3.0.a.20141031.  Not really
clear if the first came before 3.1, so opted to err on the cautious side, figuring I'd get corrected after I submitted....

Now on December 3rd, after returning from my annual trip to Chicago TARDIS, I went to work on cleaning this up for
submission....only to find that the release now has tags for v3.0 and v3.1.  The v3.1 tag appears to have occured
'one day ago'.

So, off to update my patch to this version.

Meanwhile, got my answer.... on 'zen' I had installed it as 3.1.a.20141031, while on 'dbox' as 3.0.a.20141031.  Locked the
pkg on both (even though only an issue for 'dbox', since even though I made the switch to using my poudriere repo for 'zen'
updates....I haven't had a successful build of `x11/gnome3` yet.) to protect against it getting downgraded to 2.4.2.  Well,
went to install 3.1 on zen, and portmaster stumbled on the pkg being locked and complained that the older version
3.1.a.20141031 was already installed....

Should go through `poudriere testport` without a hitch, right???

PR: 195617
