---
title: "irc/bitlbee update forces me to update irc/irssi-otr port"
layout: post
comments: yes
category: patches
---

The recent update to the `irc/bitlbee` port ran into a snag for me.  The update changes its dependency on `security/libotr3`
to `security/libotr` (*4.0.0*), where the two ports conflict with each other but `irc/irssi-otr` still depends on the older
library.  This makes it hard to use OTR when both bitlbee and irssi are on the same server, which is how I use it.

After some investigation, it appears that there had been work post 0.3 of irssi-otr that included adding libotr-4.0 support
but no release had materialized.  But, then I found a repository on github that seems to contain its history and jumps to
a v1.0.0 release (after two alpha releases.)

So, I took a stab at updating this port....and eventually got to where portmaster installed it.

I might need more work to become a real port update, but I'm starving....

PR: 192026
