---
title: "net/freeradius2: fails to build if devel/libexecinfo is installed"
layout: post
comments: yes
category: patches
---

Updating to 2.2.4 was failing for me, turns out configure has an auto-activation
on execinfo.h, except it wasn't using it correctly for FreeBSD.  So, I made it
an explicit library depend and made it link to the library.

PR: ports/188089
