---
title: update sysutils/devcpu-data with Intel microcode-20140624
layout: post
comments: yes
category: patches
---
While cleaning up some obsolete promises in a CFEngine policy, namely the
update `sysutils/devcpu-data` after its installed with the latest microcodes.
I spotted that Intel had released a new microcode bundle on June 24th.  So, I
updated this port to that microcode release.  It doesn't contain newer
microcode for my CPUs though.

With this post I got to test my hack to update my pr-handler for the FreeBSD
bugs switching from GNATS to Bugzilla.  Only one typo :blush:  The regex will
probably need some tweaking to make sure it doesn't trigger when it shouldn't
or that isn't too greedy....

PR: 191411
