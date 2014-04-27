---
title: Various issues with hplip with cups-1.7+
layout: post
comments: yes
category: patches
not_pretty: yes
---

This port's LIB_DEPENDS is testing for a library that is no longer part of
`print/cups-base` to determine if it is installed.  So, updated it to look for
a different library.  Update to 3.14.4 while I'm here.  And, patch the
deprecation warning emitted by footmatic-rip-hplip.

However, this leaves one remaining problem.  All the PPD files appear to
contain:

    *cupsFilter: "application/vnd.cups-postscript 100 foomatic-rip-hplip"
    *cupsFilter: "application/vnd.cups-pdf 0 foomatic-rip-hplip"

But, unlike foomatic-rip, foomatic-rip-hplip only knows how to handle text
or postscript.  Removing the "application/vnd.cups-pdf" line, makes it stop
trying to print PDF files as text (just the PDF header emits.)

I started working out a script that with through all the PPD files, uncompress
the files, remove the line, recompute the file size, update the file size line
at the end of the file and recompress.  And, then was going to convert the
script into something that would work from `post-patch:`

------

But, then I got busy with work and life and by the time I came back...it appears
that on April 18th, somebody else got a patch in.

Wonder if I should update my system to their version and compare?  Since I
didn't see the update because our portversions are the same....
