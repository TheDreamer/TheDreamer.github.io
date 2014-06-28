---
title: Fix build of devel/libexecinfo on amd64
layout: post
comments: yes
category: patches
---
So, I have not had a working chromium on FreeBSD since 33.0.1750.152\_1.  And, this has been really annoying.  I had even
tried recompling everything it depended on, which only made the problem worse. _Though that was partly because I had made
the mistake of turning on the __DEBUG__ option_.  All the while, everything continued to point to libexecinfo as the problem
dependency.  But, all the rebuild/reinstalls of it didn't make any difference.

But, then I happened to look at its Makefile, before rebuilding it for the millionth time.  I then noticed that the build
output was missing a compiler flag.

The top-level Makefile has the line:

    CFLAGS_amd64= -fno-omit-frame-pointer

But, this flag is nowhere to be seen in the build output.

I poked around with `make -dv` and other `-d` options, but could not figure out where it would forget that it had added
this to __CFLAGS__ for the build.  Seems like at some point it starts over again, and its somewhere that doesn't know about
the `CFLAGS_${ARCH}` feature.

Looked at the port's Makefile, and noticed it ends with `.include <bsd.lib.mk>`.  This isn't in `/usr/ports/Mk`, it's in
`/usr/share/mk`.  Hmmm.... :confused:

Guess that's the problem... So, I added a patch to the port's Makefile to check __MACHINE_CPUARCH__ to add this flag. And,
now it shows up in the build output.

And, right after I installed this patched port, I tried to start chrome...and it worked! :grinning:

Now to use `chromium-35.0.1916.153_1` to enter a bug.

PR: 191465

