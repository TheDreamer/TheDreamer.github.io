---
title: "net-mgmt/nagios-check_dhcp.pl"
layout: post
comments: yes
category: ports
---

Somewhere along the way a `portsnap update` removed `net/p5-Net-DHCP-Watch`
from my `/usr/ports` directory, causing `portmaster -a` to abort.  Been getting
kind of annoying lately with all the reasons it'll abort or do the wrong thing
now.

It also complains (and aborts) that I have packages installed that have recently
been deleted from ports.  I haven't yet investigated an alternative to
`net/nxserver` or `net/freenx`, it still works (wasn't that long ago that I
submitted a patch for one of the components.)  But, the company has stopped
making it generally available to the public, thought there's a note about it
working with certain open source projects.  Along with it being available to
its paying customers.

Anyways back to this port, one way to fix this would be to hit send on my
submission to make `net/p5-Net-DHCP-Watch` into a real port.  But, since the
reason for its existence is this nagios plugin.  I went to work on turning
this into a port as well.  Following other ports and reading stuff in
`/usr/ports/Mk`, I did staging with this.

Will I hit send?

PR: ports/187623
