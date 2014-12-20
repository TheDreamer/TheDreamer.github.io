---
title: I'm a (bad) C Programmer, so I broke net/tigervnc
layout: post
comments: yes
category: patches
---

Back on November 19th, `x11/gnome2` and pretty much everything associated with it disappeared from ports and `x11/gnome3`
appeared.  I had just switched 'zen' over to getting pkg upgrades via my private package repository courtesy of
`ports-mgmt/poudriere` the day before.  I have a mostly working, every so often it misses a day...usually when I've been
messing around with extra bulk jobs the night before....a CFEngine driven daily mash up of ports and (my) patches, using
`ports-mgmt/portshaker` (originally installed before linux-c6 was finally able to replace linux-f10....) and then running
two bulk jobs, one after the other.

My two headless servers, 'cbox' and 'dbox', have been using their poudriere package repo for updates since late August.
Didn't get things to largely automated until November.  But, since build for servers is before build for 'zen', and its
the build for 'zen' that proved to be most challenging.  I still have a backlog of PRs to submit for things I had to
patch to build under 'poudriere' (its really holding back my workstation at work, as options for propagating my patches
are limited...though I had partially switched it over in early November.

Which is important now that 3.1 allows me to try to place higher priority on certain ports or more important allow the use
of MAKE_JOBS to speed up the build of certain ports, though the latter doesn't seem to be working for me...yet.
`editors/libreoffice` takes 8 hours to build as a single job, and since its usually the last port of a bulk job it can run
into the next day's ports tree update.  Though I have made CFEngine aware of this possibility and delay 'portshaker' merges
until the bulk job is done.  And, providing the bulk job had started the day before, the sequence will be kept.  Within
limits to prevent the whole sequence getting derailed....

Well, `x11/gnome3` was failing to build after the switch for me and there has been lots of work here and there to get it
working.  I wasn't alone in the pain.  It finally reached completion on the evening of December 11th, where I foolishly
upgraded to it and ended up with no more desktop.  Its kind of working now....no more Xinerama, though Options containing
that are still part of the configuration...related to defining which monitor is on the left.  When I finally to X up again,
the were switched and I couldn't get the options to make it right again, wasn't until I later ran `nvidia-settings` that
it generated some nvidia specific options to define monitor order/placement.  But, then gdm had its own ideas that were
opposite, so more fiddling there.  Neither are really working, but its semi-usable.  Seem gnome-shell has a memory leak and
get's itself partially killed (bottom bar) after about 48 hours, have to completely kill it and then restart it....

Later I noticed that `net/tigervnc` had disappeared from my system.  This was after I noticed that `textproc/meld` had
disappeared.  Well, it used to get installed by `x11/gnome2-fifth-toe`.  So, I update my bulk jobs lists to get it back in.
But, it is failing to build.  Which is a stumper since hasn't changed much recently.  The last change was on November 22nd,
of "Cleanup plist", and previous changes to the new 'USES=python' (Oct 24) or the switch of the default gcc (Sep 10) from
4.7.4 to 4.8.3.

I finally start scouring the Internet as to why it wasn't building when I finally found this:

{% highlight text %}
Re: [tigervnc-devel] Ubuntu trusty build failures

Alan Coopersmith Thu, 11 Dec 2014 17:34:03 -0800

On 12/11/14 05:19 PM, Brian Hinz wrote:
| As of yesterday, upstream patches to the ubuntu 14.04 xorg-server-source package
| cause our build to fail with the following error:
| 
| ../../include/regionstr.h: In function 'void RegionInit(RegionPtr, BoxPtr, int)':
| ../../include/regionstr.h:147:45: error: invalid conversion from 'void*' to 
| 'pixman_region16_data_t* {aka pixman_region16_data*}' [-fpermissive]
|               (((_pReg)->data = malloc(rgnSize)) != NULL)) {
| 
| Still looking into what changed that's now causing this, but thechangelog  
| <http://changelogs.ubuntu.com/changelogs/pool/main/x/xorg-server/xorg-server_1.15.1-0ubuntu2.5/changelog>
|   references CVEs so we should probably review them to make sure the 1.4.0 release is not affected.

Sorry, I'm a C programmer, so I'm in the habit of deleting casts of malloc()
results, forgetting that breaks C++.

After the patches were released, I also found late yesterday that this change
had broken our  TigerVNC 1.1 package build on Solaris:

-        if (((_size) > 1) && ((_pReg)->data =
-                              (RegDataPtr) malloc(RegionSizeof(_size)))) {
+        if (((_size) > 1) && ((rgnSize = RegionSizeof(_size)) > 0) &&
+            (((_pReg)->data = malloc(rgnSize)) != NULL)) {

from http://cgit.freedesktop.org/xorg/xserver/commit/?id=97015a07b9e15d8ec5608b95d95ec0eb51202acb

I can make it build again by putting the (RegDataPtr) back but was hoping we
could find some way to make extern "C" { ... } or similar convince the compiler
C code was okay, since on the upstream Xorg side we have no way of knowing when
our C changes break VNC trying to use our C code as C++ code.

--
        -Alan Coopersmith-              alan.coopersm...@oracle.com
         Oracle Solaris Engineering - http://blogs.oracle.com/alanc
_______________________________________________
xorg-devel@lists.x.org: X.Org development
Archives: http://lists.x.org/archives/xorg-devel
Info: http://lists.x.org/mailman/listinfo/xorg-devel
{% endhighlight %}

Hmmm, I was an anti-C++ C Programmer...but I have on occasion been forced to do C++....

As C++ is more strongly typed, the cast is required.  But, from an ANSI-C perspective, there are basically 3 disadvantages
do using a cast.

1. the cast is redundant, saves unnecessary typing.  _But, it was already there so removing it resulted in unnecessary typing._
2. may mask failure to include `stdlib.h`, which is a big problem if sizeof(int) != sizeof(void *).  _Are you a good C Programmer?_
3. If the type of the pointer is changed, need to fix all `malloc` uses (unless it was cast to a `typedef`). _`RegDataPtr` is a `typedef`._

And, there's if its not broken, don't break it.....the fix is to add an additional test before the `malloc`, is not a reason
to mess with the `malloc`.  And, then there just seems a whole lot of other things wrong with what was said in the message.

So, I whip up a patch to `x11-servers/xorg-server/files/patch-CVE-2014-8092-3-4` and wait to see if this results in a
successful build of `net/tigervnc` in _poudriere_.

It is now about a quarter past midnight....I start to submit the PR, I first check to see if anybody else had beat me
to the punch.  Seems the maintainer of `net/tigervnc` has submitted a patch, which was to rollback the specific part of the
CVE-2014-8092 patch around 7:30pm.  Being some what angry about the whole thing and it being so late, I decided to submit
my PR (rather than add to PR: 196118)

----------

Later on the 19th, `x11-servers/xorg-server` is updated to 1.14, and I see that `patch-CVE-2014-8092-3-4` contains the
fix.  As to the switch from HAL to DEVD, that will probably require some tweaks to the CFEngine policy that takes care of
that...though I'm still working through the issues with `x11/gnome3`, while contemplating switching to `x11/mate` or checking
out `x11/cinnamon`.  Or perhaps XFCE, which is what I'm running on my remaining Ubuntu desktop (it mainly exists to run the
BOINC projects [Radioactive@Home](http://radioactiveathome.org) and [Quake-Catcher Network Sensor Monitoring](http://qcn.stanford.edu/sensor),
though there's also an NVidia card present so it is the only system I have left doing GPU work.)

Hopefully this big rebuild of ports goes smoothly, and I can get back to seeing about another annoyance I have with a port
from my poudriere repo.

PR: 196119
