---
title: extra patch to get gnucash to build for me after the webkit-gtk2 update
date: 2015-01-14 06:17:00
layout: post
comments: yes
category: patches
---

On Dec 25th, 2014, `www/webkit-gtk2` was updated to 2.4.7.  This lead to a number of ports failing to build due to its use of
__compiler:c++11-lib__.  And, since I had waited to after Christmas to finally get around to updating ot 9.3, which required a
rebuild of all my ports, this was preventing me from reach that target until late on January 8th, 2015.  There was a patch to this
port (on December 30th, specifically for 8.x but also applicable to 9.x) to also use that, but it wasnt' sufficient to get things
to build for me.

After lots of trial and error, I eventually settled on injecting `LD_LIBRARY_PATH=/usr/local/lib/gcc48` into the *MAKE_ENV*.

Wonder if I should wait to see if everything still builds with the update of `www/webkit-gtk2` to 2.4.8?

----

Turns out it causes another set of ports needing attention...which was part of the delay in submitting, the other being that the
bugs site was down.

PR: 196704
