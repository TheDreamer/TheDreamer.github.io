---
title: Getting chinese/arphicttf to build in poudriere
date: 2015-01-14 06:07:00
-description:
layout: post
comments: yes
category: patches
---

For the longest time, when I started running poudriere, `chinese/arphicttf` would fail leading to `misc/freebsd-doc-en` getting
skipped.

Eventually, I decided it would be nice if I were able to get `misc/freebsd-doc-en` updated on my computer.  So I dig into where
the problem is, and find that it was with the _Makefile.ttf_ under `chinese/ttfm`.  This *'Makefile'* is not used by the
`chinese/ttfm` port, but rather it is for ports that depend on this port to use during their builds.

Seems the post-install isn't applicable to PACKAGE_BUILDING, though it seems to me its also not applicable for staging in general.
Then there were some complaints from poudriere that the package was trying to remove things it didn't create.

Now I finally have my own copy of the latest `misc/freebsd-doc-en` from poudriere :triumph:

PR: 196702
