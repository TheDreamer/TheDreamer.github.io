---
title: "patching audio/gsm"
description: "Got a strange error, while troubleshooint something else likely unrelated, about /usr/local/bin/tcat and /usr/local/bin/untoast.  Examining the files, I found: ..."
layout: post
comments: yes
category: patches
---

Got a strange error, while troubleshooint something else likely unrelated,
about `/usr/local/bin/tcat` and `/usr/local/bin/untoast`.

Examining the files, I found:

    lrwxr-xr-x    1 root      wheel            5  2 Jul 21:27 tcat@ -> untoast
    lrwxr-xr-x    1 root      wheel            5  2 Jul 21:27 untoast@ -> untoast

Yeah, that doesn't look right.

Rebuilt/reinstalled port.

Same result:

    lrwxr-xr-x    1 root      wheel            5 19 Jul 11:24 tcat@ -> untoast
    lrwxr-xr-x    1 root      wheel            5 19 Jul 11:24 untoast@ -> untoast

But, the files under **STAGEDIR** are correct:

    lrwxr-xr-x    1 root      wheel            5 19 Jul 11:24 tcat@ -> toast
    lrwxr-xr-x    1 root      wheel            5 19 Jul 11:24 untoast@ -> toast

Oh, pkg-plist is causing them to be wrong.

PR: 191971
