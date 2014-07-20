---
title: "removal of USE_AUTOTOOLS broke build of security/openconnect"
layout: post
comments: yes
category: other
---

This morning ran into an update to `security/openconnect` removed USE_AUTOTOOLS, which broke the build for me.

{% highlight text %}
===>  Building for openconnect-5.03_4
Making all in www
gmake[1]: Entering directory `/usr/ports/security/openconnect/work/openconnect-5.03/www'
 cd .. && /bin/sh /usr/ports/security/openconnect/work/openconnect-5.03/missing automake-1.13 --foreign www/Makefile
/usr/ports/security/openconnect/work/openconnect-5.03/missing: automake-1.13: not found
WARNING: 'automake-1.13' is missing on your system.
         You should only need it if you modified 'Makefile.am' or
         'configure.ac' or m4 files included by 'configure.ac'.
         The 'automake' program is part of the GNU Automake package:
         <http://www.gnu.org/software/automake>
         It also requires GNU Autoconf, GNU m4 and Perl in order to run:
         <http://www.gnu.org/software/autoconf>
         <http://www.gnu.org/software/m4/>
         <http://www.perl.org/>
gmake[1]: *** [Makefile.in] Error 127
gmake[1]: Leaving directory `/usr/ports/security/openconnect/work/openconnect-5.03/www'
gmake: *** [all-recursive] Error 1
*** [do-build] Error code 1

Stop in /usr/ports/security/openconnect.
*** [stage] Error code 1

Stop in /usr/ports/security/openconnect.
{% endhighlight %}

Adding some of what was removed:

{% highlight makefile %}
USE_AUTOTOOLS= aclocal automake
ACLOCAL_ARGS=  -I .
AUTOMAKE_ARGS= --add-missing
{% endhighlight %}

was enought to get the build to work again.

PR: 191705

~~~~~

The problem as it turned out was that a patch was being applied to Makefile.am trigger the need for autotools, and applying
the patch to Makefile.in makes USE_AUTOTOOLS unnecessary.

Someday I'll get around to running it.  I had a working configuration under Ubuntu, but the configuration is untested on
FreeBSD.  The main question is whether my FreeBSD installation will cope with the certificates involved.

I also wonder if all of the `sysutils/vpnc-scripts` work under FreeBSD, the port only patches one of the three scripts (the
most common one... where I'm interested in using one of the other ones.)  Someday when I have my time machine, I might
get around to it.
