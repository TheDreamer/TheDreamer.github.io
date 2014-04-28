---
title: "docs/178818: gmirror(8) says to use rc.early which is no longer available"
layout: post
comments: yes
category: patches
---

gmirror(8) has NOTES section on doing kernel dumps to gmirror providers and
wanting to use a different balance algorithm through use of /etc/rc.early and
/etc/rc.local.

But, /etc/rc.early is no more.'

The proposed fix is to automate this in the dumpon script, which had one small
flaw, which I corrected.  And, then tried to recall how to include a patch
that won't get munged by mail client.... [http://lkc.me/5D](http://lkc.me/5D)

PR: docs/178818
