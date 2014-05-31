---
title: deskutils/recoll was updated to 1.19.13, but ran into problems updating
layout: post
comments: yes
category: patches
---

On May 7, 2014, this port was updated to version 1.19.13 (previous was 1.19.10.)  I didn't get around to try to update
until around May 18.  Where there were problems with it trying to install build dependencies that were already installed.
Fixing the `*_DEPENDS` lines to look for files that exist in the port fixed that.  Next there was a problem in `pkg-plist`,
which hadn't been updated from the previous and didn't reflect the comment about getting working with STAGING.  The commit
message gives a clear pointer to the problem with `pkg-plist`.

{% highlight text %}
 - Update to 1.19.13, announce message is here:

     http://www.lesbonscomptes.com/recoll/release-1.19.html

     - Remove BROKEN
     - Remove *.pyc, not at moment fixable
     - Use STRIP_CMD only when is used PYTHON Option

     Reviewed by:        upstream
{% endhighlight %}

The build removes `*.pyc`, but `pkg-plist` wasn't updated to remove references to these files.

-----

Well, its May 26, 2014, and I still haven't gotten around to submitting my PR on this.  But, while researching for this
post, I found that on May 24, 2014.  This commit message was added for the port.


{% highlight text %}
Mark BROKEN: Fails to package

===>  Building package for recoll-1.19.13
pkg-static:
lstat(/wrkdirs/usr/ports/deskutils/recoll/work/stage/usr/local/lib/python2.7/site-packages/recoll/__init__.pyc):
No such file or directory
pkg-static:
lstat(/wrkdirs/usr/ports/deskutils/recoll/work/stage/usr/local/lib/python2.7/site-packages/recoll/rclconfig.pyc):
No such file or directory
*** [do-package] Error code 1

Reported by:    pkg-fallout
{% endhighlight %}

Guess I'll get this patch submitted, though don't think I'll update it to be based off the addition of BROKEN....

PR: ports/190295

-----

May 30, 2014 - noticed that `deskutils/recoll` was updated yesterday, but it didn't look fixed.  Seems 'maintainer' ignored the dependency part of my patch, and then set the port to having no maintainer....

Time to patch it with CFEngine, I suppose.... Getting to be quite a lot of ports getting patched on my systems by CFEngine, wonder if I need to break up my __freebsd__ policy bundle?
