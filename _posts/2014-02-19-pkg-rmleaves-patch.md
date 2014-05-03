---
title: "pkg_rmleaves always says to resize termainl to at least 80x24"
description: "Everytime that I've tried this program, it always fails with 'Dialog Error, try to resize your terminal to at least 80x24.'  But, my terminal window is 80x24, and changing the size doesn't make any difference.  Finally, I looked deeper..."
layout: post
comments: yes
category: patches
---

Everytime that I've tried this program, it always fails with:

    Dialog Error, try to resize your terminal to at least 80x24.

But, my terminal window is 80x24, and changing the size doesn't make any
difference.  Finally, I looked deeper into the problem, and its having trouble
with package comments that contain quotation marks.

So whipped up an ugly kluge to deal with them, and now its working... :smirk:

PR: ports/186904

-----

Between this and pkg_cutleaves, not sure which I prefer better.  pkg_rmleaves
being somewhat flashier with its Dialog interface, versus more features in
pkg_cutleaves?  Though pkgng's 'pkg query' and some of my own scripting/aliases
is taking a knock at both.
