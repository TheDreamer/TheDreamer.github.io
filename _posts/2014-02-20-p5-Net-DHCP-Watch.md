---
title: "Net::DHCP::Watch - A class for monitoring a remote DHCPD server."
layout: post
comments: yes
category: ports
---

Couldn't get check_dhcp to work on FreeBSD, so had nagios invoke it through
nrpe on a Linux server.  But, I'm running out of old Linux servers.

So, I switched to usinga check_dhcp perl script, which among the required
modules was this one.  I CPAN'd it at first, but BSDPAN is not pkgng aware.
So, decided to whip up this port.

PR: ports/187622
