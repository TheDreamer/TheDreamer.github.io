---
title: "gnome-keyring-daemon: couldn't allocate secure memory"
description: "Keep seeing the annoying message of \"gnome-keyring-daemon: couldn't allocate secure memory to keep passwords or keys from being written to the disk\" on my FreeBSD system.  And, it continues even after I had set \"security.bsd.unprivileged_mlock=1\" back on December 20, 2013.  The default resource limit RLIMIT_MEMLOCK is 64k, which I would think is more than sufficient."
layout: post
comments: yes
category: patches
---

Keep seeing the annoying message of _"gnome-keyring-daemon: couldn't allocate
secure memory to keep passwords or keys from being written to the disk"_ on
my FreeBSD system.  And, it continues even after I had set
"`security.bsd.unprivileged_mlock=1`" back on December 20, 2013.

The default resource limit RLIMIT_MEMLOCK is 64k, which I would think is more
than sufficient.

So, it was time to research this problem in more depth.

Found that there's a `DEBUG_SECURE_MEMORY` define to see how much memory its
trying to allocate.  Which is trying to allocate blocks of 16k, which it later
refers to as pages.  Thinking that its Windows that did memory pages that big?
Because most Unix/Linux systems use 4k (including my FreeBSD system), except
for Solaris which is 8k.

I tried change the constnat to 4k, but this also failed.

I had skimmed the `mlock(2)` man page, where it had said:

> > Since physical memory is a potentially scare resource, processes are limited
> in how much they can lock down.  A single process can mlock() the minimum of
> a system-wide ''wired pages'' limit vm.max_wired and the per-process
> RLIMIT_MEMLOCK resource limit.
>
> > If security.bsd.unprivileged_mlock is set to 0 these calls are only
> available to the super-user.

Well, on my system vm.max_wired is 1323555 and RLIMIT_MEMLOCK (`ulimit -l`) is 64 (kilobytes.)

_Wrong_.  Delving into the Kernel source, I found that it first checks that
the per-process limit, and then that the requested amount + the amount already
wired system wide (r/o sysctl `vm.stats.vm.v_wire_count`) is not greater than
`vm.max_wired`.

Well, when I looked at `vm.stats.vm.v_wire_count, it was 2020311... its already
got more than `vm.max_wired` wired! :scream:

I feel a PR coming on....

132555 (which is about 5GB) is said to be 1/3 of some maximum...its a 16GB
system, probably some number less than contiguous to allow paging & I/O.
2020311 is about 7.7GB.

So, I did a "`sysctl vm.max_wired=2097152`" (8GB), and it took it (so I added it
to `/etc/sysctl.conf`, too.) and now __gnome-keyring-daemon__ can start without
that message.

PR: docs/189214
