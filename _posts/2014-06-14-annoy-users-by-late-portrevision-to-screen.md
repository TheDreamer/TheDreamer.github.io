---
title: "Let's annoy the users of screen again"
layout: post
comments: yes
category: patches
---
So, back on April 30th, 2014, the `sysutils/screen` port was updated from version 4.0.3_14 to 4.2.1.  One of the annoyances
that resulted from this was that existing 4.0 screen sessions became inaccessible.  There was a portrevision on May 1st for
4.2.1_1 to fix colour in hardstatus (upstream patch) and remove some options that could get patched into 4.0 (which appear
to be part of 4.2.1 now), and later without a PORTREVISION bump on May 23rd to add LICENSE.  But, then early this morning
(for a PR entered late afternoon yesterday) to change the socket.c code to use FIFOs instead of sockets, to be backward
compability with what 4.0's configure had selected.

After digging around in screen's git repository, and then examining the contents of the screen-4.0.3.tar.gz source archive.
It appears the problem was an error in how this archive was generated.

4.0.3 was born on October 23rd, 2006.  With a commit on December 19th, 2005 that changed it to prefer sockets over fifos,
when both are available.  However, the screen-4.0.3.tar.gz archive's `'configure.in'` has a timestamp of June 3rd, 2003.
Corresponds to the screen-4.0.0.  The first commit changing it wasn't until just before the change to favor sockets, and
well after what was 4.0.2.  But that first commit was a security bugfix (wonder if 4.0.3 actually contained the fix without
the right configure?)

While I had already gone through the pain of nuking my old screen 4.0 session, right after I had upgraded to 4.2.1.  It
seems odd that this was an issue requiring a PR yesterday.  Since between the appearance of 4.2.1 and yesterday, my core
servers had been rebooted at least twice.  The first time to take it from '9.2-RELEASE-p5' to '9.2-RELEASE-p6', and the
second time took it from '9.2-RELEASE-p6' to '9.2-RELEASE-p8'.  I had thought it was only going to '9.2-p7', since I done
the update to '9.2-p7' on my home workstation.  But, there were only a few days between '9.2-p7' and '9.2-p8', such that
when I did get to doing my core servers, they went to '9.2-p8'.  Though there was no kernel change for '9.2-p8', so uname
stayed with '9.2-RELEASE-p7'.  Didn't notice that there was a '9.2-p8' until I got around to updating work workstation,
which I had been holding off on, while it was in a degraded state having lost a drive....  Going back to my systems at home
I only had one machine that needed new files to bring it to '9.2-RELEASE-p8'

In the source, there's a file called 'socket.c', that depending on the definition of __NAMEDPIPE__, selects beween the
mutually exclusive support of Unix domain sockets or named pipes (FIFOs.)  So, I suppose logically sockets should be the
preferred option.  Appears 'socket.c' came into existence with screen-3.1 (with conditional using __NAMEDPIPE__.)  Wonder
what the file was named before this change.  Appears that prior to this it was just one huge source file named 'screen.c'.

Oh well, if this old behavior is still desired, it should be as an option, though then whether it should be the default
is probably up for debate.  The patch I made is that its a option where it is non-default.

PR: ports/191029
