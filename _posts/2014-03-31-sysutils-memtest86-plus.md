---
title: "systutils/memtest86+: Doesn't build if ISO option is selected"
layout: post
comments: yes
category: patches
---

Latest update doesn't build if ISO option is selected, the distribution Makefile
is calling make but the port is using gmake.

Patch the Makefile to use gmake.

PR: ports/188149
