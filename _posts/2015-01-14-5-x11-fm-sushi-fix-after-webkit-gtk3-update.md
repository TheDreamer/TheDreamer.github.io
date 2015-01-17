---
title: x11-fm/sushi fix build after webkit-gtk3 update to 2.4.8
date: 2015-01-14 06:21:00
layout: post
comments: yes
category: patches
---

Well, the update of `www/webkit-gtk2` & `www/webkit-gtk3` to 2.4.8, results in the latter now also *USES* `compiler:c++11-lib`.
Which causes this port to fail until a modification similar to getting ports affected by `webkit-gtk2-2.4.7` is done.

This was first one that I tackled, because of its name....

PR: 196705
