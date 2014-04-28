---
title: xpi/accountcolors updated
layout: post
comments: yes
category: other
---

I failed to notice that around January 18th, 2014, that this extension had
updated to 6.11.  Probably because things had continued to work, and I didn't
make any changes.

The code had bumped limits from 60 to 160, but the options dialog was still
limited to 60 rows.

I applied my patch to bring the options dialog to 128 rows.

-----

A couple hours later, 7.0 was released.

The code was back to a limit of 60, so I patched it all back to 128.
