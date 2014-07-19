---
title: "[new port] www/p5-LWP-Protocol-connect: provides HTTP/CONNECT to LWP::UserAgent"
description: "Seems to me that proxy support for connecting to https sites with LWP::UserAgent had been broken forever.  But, then I get asked to install a commericial product's perl script into one of our nagios instances, and its relying on LWP::UserAgent's proxy method to work correctly when POST'ng to its https site.  Or, it still thinks people only run such sites were direct access to the Internet is always possible."
layout: post
comments: yes
category: ports
---
Seems to me that proxy support for connecting to https sites with
LWP::UserAgent had been broken forever.

But, then I get asked to install a commericial product's perl script into one
of our nagios instances, and its relying on LWP::UserAgent's proxy method to
work correctly when POST'ng to its https site.  Or, it still thinks people only
run such sites were direct access to the Internet is always possible.

I tried various hacks to get the script working, while trying to avoid
rewriting it as little as possible.

After trying what I thought was similar to what I had done once before, but
from memory as the machine has long expired and I didn't have a way to access
its archive (or indication that the archive would have what it.)  And, it was
late, so I was tired and hungry....so all the attempts were doomed.

In the end, the next day...I eventually settled on trying
LWP::Protocol::connect, the first of many different suggestions online, to
overcome something has been broken for over 10(?) years in LWP::UserAgent,
that worked!

There was even an new version of LWP::UserAgent released on July 1st, 2014 (the
previous on April 16th, 2014).  But, updating all the Perl modules that updated
between the time I had installed the product and when it stopped working didn't
help.
        
Though the application user said it had been working, but stopped on around the
4th of July.  Later admitting that they had changed delivery method from email
to POST.  Thinking that POST would be more direct.  Except that email means
sending through one of a pair of smtp servers behind our F5.  And, POST
involves using one of a pair of proxy servers behind our F5.  

The smtp servers get regular attention, while the proxy servers which had been
abandonned on us to provide haven't gotten any attention since they were setup
some 7 years ago.  Though we're considering upgrading the F5's to 11.5.1 and
using its in-built proxy capabilities.

Though now that its working, it probably avoids any network issues that would
cause the message to queue on our smtp servers.  Instead remaining in its queue
on our nagios server.

Where before it was working was starting an instance that would block all
future instances, and it runs every minute from cron, until it eventually runs
out of memory/swap and things stop. (though it stopped early, because
kern.maxswzone hadn't been adjusted to reflect the size of the swap volume...)

Meanwhile, made a one line change to the applications perl script for

    $ua-proxy('https', $opt_proxy);

And, then invoke the application with
`--proxy=connect://<our-proxy-server>:8080/'

When I tried it by hand, it worked immediately.  So, I made it permanent.

The reason I resisted trying this in the beginning, was that it wasn't in ports
and I hadn't heard whether BSDPAN had been updated to support pkgng.
        
(Bug #187111, reported February 27, 2014 - only one comment, dated March 20,
2014, from somebody saying its a blocker for their client, where client is
willing to pay.)

So, I had immediately thrown together a port for it.  Somehow it worked with an
invalid pkg-plist (forgot to update it from the port I had used as my
template..._p5-LWP-Protocol-https_)....wonder what happens if I were to try to
uninstall it?

Cleaned up now, guess its time to submit it?

PR: 191784
