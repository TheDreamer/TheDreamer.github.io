---
title: fix build of x11-toolkits/wxgtk30 after webkit-gtk2 update
date: 2015-01-14 06:12:00
layout: post
comments: yes
category: patches
---

Related to the Dec 25th, 2014 upgrade of `www/webkit-gtk2`, this port was also failing to build and resulting in a number of
skipped ports.

On January 4th, 2015, I noticed the change to `finance/gnucash`'s Makefile, so I tried that addition here as well.
Eventually, found that I needed to make one additional change.  In addition to this and the upcoming entries, there had been two
other ports failing.  But, by the time I get this one fixed, the only one still failing was `finance/gnucash`.  Didn't look to
see if the other two had been related to this, or if they had had some other problem.

The one additional change was to add `--disable-precomp-headers`, didn't really want to chase down why this had apparently worked
with base gcc but not with gcc48.

PR: 196703
