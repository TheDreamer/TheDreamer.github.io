---
title: Test4Theory keeps telling me that my VirtualBox is missing
layout: post
comments: yes
category: patches
---

When I first started running boinc on FreeBSD, I discovered to my delight that
VirtualBox was detected and usable for Test4Theory.  But, somewhere along the
way, when I checked on things, it would report that it didn't detect VirtualBox.
This was usually resolved by restarting it, as this was usually some contention
between the respective rc sripts.

Except that I was seeing it more and more lately, but having solved the rcorder
problem I was having (where `/etc/rc.d/tmp` and `/etc/rc.d/cleartmp` were being
called after `/usr/local/etc/rc.d/vboxheadless` -- partly my fault because I
run an NFS server in the headless VM which my host mounts.  Its how I get
dropbox into my FreeBSD system.)

But, the problem continued to happen with `boinc-client`.  So, I decided to take
a closer look.

I found that when `hostinfo_unix.cpp` thinks its a linux like system, it looks
to run **`/usr/lib/virtualbox/VBoxManage`** (which is located at
**`/usr/local/lib/virtualbox/VBoxManage`**, of course, on FreeBSD.)  Else if
its `__APPLE__`, it looks the app, etc.

So, I added a patch to handle `__FreeBSD__`.

Not sure when or how this got broke.  At first I thought a patch had
disappeared, but didn't find anything resembling my patch (though I only went
back a couple of years...), more likely support for VirtualBox on FreeBSD
disappeared in the 7.x version of the code.

Strange how I had failed to notice for so long....

PR: port/188710
