---
title: "databases/memcached: install error with or without DOCS"
layout: post
comments: yes
category: patches
---

It is becoming a peeve of ports that work fine, but PORTREVISION is bumped so 
that it will disrupt "portmaster -a" or disrupt my system.  Especially when it
causes portmaster to remove the working build and fail during the install of
the bumped port.  Especially, when the bump is of questionable importance,
though it might only be 'especially' since it broke something that wasn't
really broken.  Had it not broken things but was an enhancement...well, hard
to say in this case.

Here's the patch I made to make a new port option work.  Both for my 'servers',
where I have "OPTIONS_UNSET = DOCS" set in /etc/make.conf, and on my desktop
where I let defaults take affect.

PR: ports/187337

There were two other PR's about problems relating install errors.  PR/187309
with DOCS and PR/187340 without DOCS.  First only reported the failure, the
latter provided a patch similar to mine.  The final patch wasn't like either.

Discovered later that I couldn't get into my email, to see what number had been
assigned.  Roundcube lobs have "DB Error: Failed to connect to memcached."
Also noticed that memcached is consuming 100% of CPU.

Backing out the above referenced PORTREVISION, to "Enforce libevent2" patch,
got things working again.  The maintainer reworked his patch to work if
both libevent1.4 and libevent2 were on the system.

Where libevent1.4 was installed as dependency of previous install of memcached, and libevent2 was installed as dependency for tor. (but if it wasn't for tor,
it would've been installed as dependency of new memcached, and the old
doesn't get removed unless I were to run something like pkg_rmleaves.)

On my desktop, chromium depends on libevent1.4, while firefox/thunderbird/libxul
depend on libevent2.
