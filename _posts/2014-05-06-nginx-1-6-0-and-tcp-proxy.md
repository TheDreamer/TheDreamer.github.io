---
title: www/nginx updated to 1.6.0, TCP_PROXY disabled
description: "Yesterday the port updated from 1.4.7 to 1.6.0, but seems a number of 3rd party modules were disabled rather than the maintainer tracking down updates for them.  I use TCP_PROXY..."
layout: post
comments: yes
category: patches
---

Yesterday the port updated from 1.4.7 to 1.6.0, but seems a number of 3rd party
modules were disabled rather than the maintainer tracking down updates for
them.

I use TCP_PROXY, so I was impacted.

I tracked down a fork of this 3rd party module that had been updated to work
with nginx-1.6.0.

After extremely minimal testing (it compiled :wink:) on dev, I deployed it to my
'production' servers.  Though the application that needs it hasn't gone into
production yet...but will hopefully later today. :stuck_out_tongue_closed_eyes:

PR: ports/189393
