---
title: mail/dovecot2 doesn't detect libstemmer or exttextcat
layout: post
comments: yes
category: patches
---

Wanting to try out the FTS plugin in dovecot2, found that it was failing to
find libstemmer/exttextcat libraries.

Found that `textproc/libexttextcat` has incremented its library name, so
configure has trouble finding it.

libextextcat has gone from:

    libextextcat.so(.*) -> libextextcat-1.0.so(.*) -> libextextcat-2.0.so(.*)

As for libstemmer, this is present in lib-clucene-contribs-lib.so from
`textproc/clucene`, so patched the configure & code to look for it from there.

PR: ports/175813
