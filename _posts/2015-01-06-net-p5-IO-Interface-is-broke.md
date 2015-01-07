---
title: p5-IO-Interface-1.09 is broken
description: "So, after yesterday's knee jerk patch to net-mgmt/nagios-check_dhcp.pl turned out to be a failure.  Which came about because I had failed to notice that I hadn't updated `net/p5-IO-Interface` on my test host.  It had only been updated in my jails..."
layout: post
comments: yes
category: other
---

So, after yesterday's knee jerk patch to `net-mgmt/nagios-check_dhcp.pl` turned out to be ~~a failure~~_unnecessary_.  Which
came about because I had failed to noticed that I hadn't updated `net/p5-IO-Interface` on my test host.  It had only been
updated in my jails...

Started into a deeper examination of `net/p5-IO-Interface`, saw the patch to fix a bug in 1.06, looked at the corresponding
code in 1.09 (instead of a while loop that changes the pointer, and then trying to free at the point it had descended to.
Though in my reproducing the section of code as a standalone test, it was the first entry that yielded the desired result
so probably something that was easily overlooked.  The patch file created a save pointer which was used late when it was time
to 'free' the data.  Using a for loop, obviously, results in having an index that moves along the linked list leaving the
original pointer unchanged for 'free'ing.

So, started wonder if there was a compile problem, tried various incantations to introduce different compiler flags to no
avail.  Not familiar with `Module::Build`, so no idea on where to fiddle with it.  But, it occurs to me to wonder if its
evening using the right section of code in *Interface.xs*.  So, I inject an intentional typo.  And, it compiles without a
hitch.  So, I then wonder where does `'HAVE_SOCKADDR_DL_STRUCT'` and `'USE_GETIFADDR'` get defined so that it will use
the correct section of code?

It was in the *Configure* section of `Makefile.PL`, no idea how to get that into `Build.PL`.  So, update my bug report and
write this post....and roll back the `p5-IO-Interface` (first use of `'KEEP_OLD_PACKAGES'` in my poudriere setup) and lock
it against updates....

-----

Last thing is re-enable DHCP monitoring from the two places that I'm doing it with `net-mgmt/nagios-check_dhcp.pl`.
