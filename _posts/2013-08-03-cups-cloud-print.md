---
title: print/cups-cloud-print
layout: post
comments: yes
category: ports
---

While research options for replacing my old inkjet printer, and the apparent
obsession of vendors to do GDI and Windows only... though some support Mac OS X
and some might add some Linux support.

Since I already ran into this mistake with my laser printer, which I hardly use
now.  Though I run a small ubuntu VM on one of my FreeBSD systems to make use 
of it.  It appears unlikely that I'll have Windows (full-time) again, and no
idea when a (full-time) Mac will become a reality.

One of the draws to the idea of working for a University was to get discount
Apple hardware, but I've yet to get anything this way in over 7 years.

I had seen printers with direct Google Cloud printing support, but the
capability had disappeared from Chromium on FreeBSD some time ago.  I do have
the capability to print to my printers at home, since I own a Chromebook.
Again it involves running chrome in that small ubuntu VM.

So, I had gotten a version of [cups-cloud-print][1] to build and run.  And, I
successfully printed a web page from my browser on FreeBSD, out to Google, back
to chrome running in VM on same computer, to printer sitting next to said
computer.  Though probably wouldn't be any less weird if I did get a printer
with direct cloud printing support.

The port probably needs a lot more work to get it to FreeBSD standards, but this
work stalled on trying to figure out how `MASTER_SITES=GH` works.

Not sure I remember what I did anymore, so thought I should save my work
somewhere before it got lost.

[1]: http://www.niftiestsoftware.com/cups-cloud-print/

------

**2014-02-04** (first snow day of 2...so far): Was about to take another stab
at getting it to work, when I noticed that an official port for it had appeared
with timestamps of `2014-01-29`.  Wonder if I want to switch to it.  Though my
current install was not installed by port, so that might cause problems.
Noticed that Makefile operates on `/etc/cloudprint.conf` (which is where my
local install works on, but in files I had patched the appropriate files to
use /usr/local/etc....
