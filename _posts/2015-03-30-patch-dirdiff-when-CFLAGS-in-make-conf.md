---
title: patch sysutils/dirdiff when CFLAGS is set in /etc/make.conf
layout: post
comments: yes
category: patches
---

Since meld got removed during upgrade to gnome3, which I have reinstalled from ports since _it's replacement_ doesn't do
diffs of directories.  And, I often have this need.  So, in looking for alternatives, this was the first obvious one to try.

Cheating, I first tried to build this outside of _poudriere_.  Well, failed.  After some inspecting, concluded the issue was
because I have CFLAGS set in `/etc/make.conf`.  Since this file is promised by CFEngine, it takes some extra effort to
temporarily disable it (haven't yet gottent COWBOY mode to work consistently, though depending on what part of the hour it
was stopping cf-execd is one option.  cron checks if its running once an hour, and starts it up again if it isn't.

Anyways, looking in work dir, saw that Makefile was getting editted, such that the build could pass CFLAGS through its
environment.  Instead of editting CFLAGS in the Makefile to be conditional, so that passing through environment would be
possible.  Decided that the solution is that it should patch CFLAGS to be what it wants to pass in to the port build.

Intention is that _poudriere_ would later build the port and I would reinstall from there.  Of course, I had only made
the change directly to the Makefile in `/usr/ports/sysutils/dirdiff`, without making a copy of its original state.  So, I
forgot about it.  Until I noticed that the port was failing to build in _poudriere_, d'oh!  Well, _poudriere_ has its own
ports tree, so it has the port's `Makefile` in original state.

So, I pulled the two files (appropriately) into a directory under [beastie-work](https://github.com/TheDreamer/beastie-work)
and created a patch for possible submission.  And, then set the links to feed my tweaks from here to get merged into the
ports tree that get used in _poudriere_.

Hopefully, things will work in tomorrow's attempt.

PR: 199058

-----

In the meantime, the search is still on.  Unless I figure out how to hack meld to work under Gnome 3, or whatever component
that is the issue.  The current version says it needs GTK+ 3.6, and in development is for GTK+ 3.12,  But, FreeBSD has
GTK+ 3.14...  Or, that it requires Glib 2.34 now, with development for 2.36, but FreeBSD has 2.42...

Had already looked at the [DarkThemes](https://wiki.gnome.org/Apps/Meld/DarkThemes) page in attempt to come up with usable
colors.  Today, I spotted the [GConfWorkarounds](https://wiki.gnome.org/Apps/Meld/GConfWorkarounds) page, but all that got
me was a message that `Using a flat-file settings backend is not yet supported`.  The meldrc.ini that already exists was
from before Gnome3, etc.

:confounded:
