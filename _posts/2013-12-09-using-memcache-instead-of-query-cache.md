---
title: "Using memcache instead of query cache in mysql?"
layout: post
comments: yes
category: patches
---

Keep seeing the advice to use memcache instead of query cache in mysql, but
it stumped me on how would I do this without requiring that I change the
applications (the big one being cacti.)

When I finally came across this extension (`databases/pecl-mysqlnd_qc`), which
sounded like it would fit.  
Except it didn't, because I use spine. :cry:

Anyways, while trying it out, I couldn't figure out how to get it to use
memcached.  Well, these are enabled at compile time, and the port didn't
ask if I wanted to enable MEMCACHE or sqlite storage handlers.  So fiddled
with the Makefile to have it ask.

Wonder if I'll submit it?

Decided to first remove references to APC.

**`patch-php_mysqlnd_qc.c`**

Going with mysqlnd_qc.cache_by_default, there needed to be a way (to me) to have
it use MEMCACHE by default.  I started trying to add a new INI option to do
this, but got lost trying to figure out how, so I went with the QAD way....
