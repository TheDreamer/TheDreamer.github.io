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

------

**2014-06-08**  I had snuck past me that on June 5th, this port was updated to 3.8.1, but caught my attention today that
it was being updated to 3.8.1_1.  The patch I had done is still missing, but it didn't cause me any problems because a
while back I had changed the portmaster options on my home desktop to not remove build-only dependencies at the end.

Since I've been updating the bundle that patches my FreeBSD ports today, I added promises to patch the Makefile for
`math/octave` and `math/octave-forge`.  OTOH, since I'm not using any of these ports, perhaps I should just purge them
from my system.
