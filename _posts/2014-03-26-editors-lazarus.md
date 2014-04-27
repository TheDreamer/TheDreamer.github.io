---
title: Trouble building new build-only dependency editors/lazarus
layout: post
comments: yes
category: patches
---

Lately, I've been having portmaster delete all build-only dependencies, since
I merged in a patch that has it able to use pkgng to install build-only
dependencies from its local packages directory.  So, when it was time update
a port that has this as a build-only dependency, things went badly.

What I found was that when it was doing editors/lazarus-lcl-units, steps were
in need of lcl to have been built...but it wasn't going to build these until
last.

So, I patched it ot make it build sooner.

PR: ports/18761

Turns out the problem were stray files left behind by the older version.  The
maintainer says he fixed the deinstall in this version, and a later rebuild/reinstall and then delete of this build-only depend worked without my hack.

However, he didn't respond on my concern that its still only to trip other
people up until they get this update to build/install and uninstall....oh well.

