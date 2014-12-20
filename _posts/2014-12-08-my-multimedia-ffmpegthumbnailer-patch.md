---
title: Was too late getting in my patch for multimedia/thumbnailer
layout: post
comments: yes
category: patches
---

Back on December 1st, this updated port failed to build for me.  After a quick head scratch, I figured out the next
~~day~~night that adding `compiler:c++11-lang` to **USES** would make it build.

But, didn't get around to looking at submitting it until late on December 8th.  Found somebody had submitted about 4.5
hours after I had made my patch to fix build on 8.4, followed by a report today that it fixes the problem on 9.3.  I'm still
on 9.2, should maybe see about upgrading soon.

One difference though is the submitter used `compiler:c++11-lib` and patched a header file.  Wonder which is right?

PR: 195600
