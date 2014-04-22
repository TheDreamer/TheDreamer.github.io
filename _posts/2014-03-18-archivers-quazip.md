---
title: "archivers/quazip: fails to build if previous is already installed"
layout: post
comments: yes
category: patches
---

This port got updated, but it wouldn't build.  After some investigation, it was
obviously a problem of conflect with installed port.  Specifically for
libquazip.so.  Instead of resorting to "make clean deinstall reinstall", I
made it hack the Makefile so to force it use the built one instead of the
installed one.

PR: ports/187735
