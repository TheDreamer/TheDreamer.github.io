---
title: "Update deskutils/gcalcli"
layout: post
comments: yes
category: patches
---

Been trying to make use of the more advanced options of this, but haven't had
much luck.  Went to see if there was a newer version, and found that the project
had moved to github and the latest tag is 2.4.2.  Seems ports have a way of
not getting updated if the site moves, and the new releases aren't posted
to the old site, etc.

I started with a quick update to the port so that I can have the latest, which
wasn't too hard since its a NO_BUILD port.  Then I thought about sending it up,
which resulted in more changes to things.  Like option knows for the 3
optional dependencies.  The default matching the one that had been listed as
a required dependency for previous package.

And, then added copying/installing of docs, which led to staging.

Haven't tried the options yet, but it did make one calendar that I couldn't see
in the previous version visible.  Under other calendars on Google, I have
various shared public calendars and one privately shared (exchange) calendar.
Which was the main reason I was fiddling with gcalcli today.

PR: ports/187619
