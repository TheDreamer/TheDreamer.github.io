---
title: "Grive: Google Drive client for GNU/Linux .. and now FreeBSD"
description: "A while back, I got grive-0.2.0 to build on FreeBSD, and use it now and then.  Having successfully submitted a new port for FreeBSD, I thought that I should try my hand at turning this work into a port...."
layout: post
comments: yes
category: ports
---

* sysutils/grive

  I got grive-0.2.0 to build on FreeBSD, and use it now and then.  And, thought
that I should turn it into a port.  But, I think I'm out of my depth on
whether this is even right.  Plus I don't think I want to be stuck
maintaining this.

* sysutils/grive-devel

  I took a look at the current work on grive-0.3.0 on github, but it needs at
least 0.10 of devel/json-c and ports stops at 0.9.  I had started work on
making a patch to update the port, but sounds like development is heading
towards needing 0.11 now.

  I then ran into it needing a newer version of devel/boost-libs than currently
in ports.  Given the difficulty I've had in building updates as a dependency
of other ports, I didn't feel like tackling this update.

  Plus I still haven't figured out how to do distfiles from GitHub,
*MASTER_SITES=GH*.

  Otherwise, I might reveal my stab at print/cups-cloud-print.

* devel/json-c

  While technically a patch, it got recorded under my ports tree in my github
repository as it was related to the above work.  This patch would've brought
the port up to 0.10 (on 2013-11-09 the port jumped from 0.9 to 0.11.)

* 2014-03-09

  I happened to be poking around on github for some reason, when I saw that
there was a response to a year old issue for grive on making it compile on
FreeBSD.  Hadn't looked through issues before, but I only had two notifications
for Grive, and one had the word "FreeBSD" in it...  The added response was
that it builds fine from ports (net/grive).

  Wonder how long that had been there, wonder if there's some way to find out
when new ports are added?  Perhaps something I could do at the same time as
when I'm doing my daily portsnap?
