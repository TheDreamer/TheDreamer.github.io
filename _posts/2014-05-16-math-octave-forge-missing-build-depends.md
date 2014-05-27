---
title: In looking for something to handle DICOM files on FreeBSD, I tried to build math/octave-forge
layout: post
comments: yes
category: patches
---

I tried to build `math/octave-forge`, since under this meta-port was `math/octave-forge-dicom` which was the first port
I could track down that depended on `devel/gdcm` -- "_Grassroots DICOM library_".  But, the build didn't go so well, as
it kept failing because of some command wasn't found, or some expected library or header was missing.

Well, the Makefile for `math/octave-forge` and `math/octave` where missing details about its `BUILD_DEPENDS`.  In fact,
trying to build `math/octave-forge` had it fail because the command `octave` wasn't found.  The port should've also
probably listed it as a RUN_DEPEND.  Not to mention that later its going to want gmake and fortran.  Though `math/octave`
was going to need stuff to build first.

However, I then found a different port (`graphics/aeskulap`) that did what I wanted, so I haven't tried running this port.

So, will I submit a PR about these two ports?
