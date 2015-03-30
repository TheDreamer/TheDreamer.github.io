---
title: fix accessibility/redshift with VIDMODE option in poudriere
layout: post
comments: yes
category: patches
---

Been working on rebuilding all packages for my work computer in _poudriere_, where this was the last package that was
failing.  But, had built fine for my computer at home in _poudriere_.  Work computer uses an older NVIDIA card, though it
was relative new when started working there in 2006, so until I get upgraded I'm stuck where I constantly live with
applications complaining that RANDR extension is missing.

On, home workstation I see that my X display at home has both XINERAMA & RANDR, though the NVIDIA card in it has recently
had its support dropped (so I'm stuck with `x11/nvidia-driver-340`, but various *linux-c6* ports depend on
`x11/nvidia-driver`.)

As I recall, XINERAMA & RANDR were mutually exclusive when I was trying to get X working at work, which until recently was
also how I had built X at home.  Getting X working at home followed the work I did at getting it to work at work, at home I
had been using Ubuntu (10.04LTS).  Which I had had kept holding off on upgrading as I resisted the end of Gnome2, and has
since ceased when I borrowed its powersupply to restore use of my home FreeBSD workstation.  If it comes back, it might be
as FreeBSD 10.1 even though it is a only a Core2 Quad, and it may be desirable in the future to do _poudriere_ builds with
CPUTYPE greater than nocona/core2.

But, since then I find that my current X display at home has both XINERAMA & RANDR.

So, as a result of display limitations, I needed to build `accessibility/redshift` with VIDMODE.  And, I had originally
built it with RANDR turned off, as I did with as many ports as I could where the option existed.  Though for _poudriere_,
I've been trying to build it with both options on.  But, I kept running into a missing build dependency error.  It would
build fine outside of _poudriere_, both at home and at work, and had somehow built fine at home with VIDMODE.  Which was
baffling.

Though in deeper inspection, I found that a later build dependency for `graphics/gdk-pixbuf2` for my home environment would
pull in the build dependency for VIDMODE, and only because for one of its dependencies, `graphics/jasper`, I had turned on
the OPENGL option while at work I had left it as default.  (more likely because I tend to be more conservative on
non-default option setting at work vs home, than whether OPENGL was an option with my old NVIDIA card or not.)

PR: 199004

----------

I have been thinking of doing a full options review of all my ports, perhaps once I finish getting everything into
_poudriere_ and complete current upgrades (such as we still use 1.9 of `lang/ruby` in production, while switch to 2.0 took
less than a month after the default version switch...largely helped by the use of _poudriere_ and not having to defer to
upgrade windows and contending with higher priority projects.)  I'll introduce my hacked version of
`ports-mgmt/dialog4ports` to my work _poudriere_ server....  Ruby 1.9.3 is still the only binary avaialble for Lucid,
Precise and Trusty from the official Ruby site, though PPAs exist for the newer versions, though only built from unmodified
sources (such as one PPAs that offer Ruby 1.9.3 with 'major&nbsp;performance&nbsp;improvements' or Ruby 1.8.7 with
'major&nbsp;improvements', but Ruby 2.0, 2.1 and 2.2 are built from unmodified sources.  While FreeBSD ports removed
ruby 1.9 back on February 24, 2015.  Though I have a port tree in poudriere that is a svn retrieval at just before the
removal of gnome2, to try to avoid issues in upgrading systems still running 9.1 or 9.2 to 9.3.  Will worry about upgrading
to current ports when that is out of the way.

