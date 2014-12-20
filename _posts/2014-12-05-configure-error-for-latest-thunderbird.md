---
title: configure error trying to build thunderbird-31.3.0
layout: post
comments: yes
category: patches
---

Couple days ago an update to `mail/thunderbird` appeared, and it was failing during configure.  Results in poudriere
skipping some 20+ ports.  Which is strange since `mail/thunderbird` is a leaf port.  And, I only just got `x11/gnome3` done
in poudriere, on the fence on whether I'd through in the perl update now or later.  The finally stopper is
`editors/libreoffice` which takes too long to build, and get's killed just before my daily refresh of my ports directories.

After some digging around, concluded that two lines were in the wrong order.  So flipped them around and presto.  The lines
had always been that way, but the contents of **CONFIGURE_TARGET** changed as part of this update in `bsd.gecko.mk` such
that swaping *portbld* with *unknown* became necessary.  But, removing it from **MOZ_OPTIONS** as *portbld* needed to be
done first.

PR: 195728
