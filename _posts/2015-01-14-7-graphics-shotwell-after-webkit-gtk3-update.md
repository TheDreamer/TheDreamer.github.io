---
title: Fix graphics/shotwell after webkit-gkt3 update
date: 2015-01-14 06:30:00
layout: post
comments: yes
category: patches
---

This was the final port, I needed to get to, that broke after the update to `www/webkit-gtk3`.  Which was the last I expected to
run into, and was a little bit more complicated to resolve.  Especially, where the OPENMP option has code compiled with gcc that
links against a `'libc++.so'`, which had been a dependency of `www/webkit-gtk3` prior to its switch to being compiled with
`USES= compiler:c++11-lib`.

This does bring in to question whether using `compiler:c++11-lib` is limited to FreeBSD releases less than 10, but I stick with
something that maintains the separation.  Though a quick glance at freshports, seems to say that `www/webkit-gtk3` was the
primary consumer of `devel/libc++`.  Where **libc++** was a *c++11-lib* for use with prior gcc compilers.  And, the *USES* of
`compiler:openmp` had done the *USE_GCC* that was being done here.  And, a *USES* of `compiler:gcc-c++11-lib` would take this
further that the added **LIB_DEPENDS** for `'libc++.so'` but makes changes to other environment variables that differed from what
this port was doing.  So, opted to only switch to a *USES* of `compiler:openmp` and inserting a *LIB_DEPENDS* for go with the
existing *LDFLAGS* mod.

{% highlight make %}
# because webkit-gtk3 needs it
.if ${OSVERSION} < 1000000
USES+=		compiler:c++11-lib

.if ${PORT_OPTIONS:MOPENMP}
LDFLAGS+=	-lstdc++
.endif

.else
.if ${PORT_OPTIONS:MOPENMP}
USES+=		compiler:openmp
LIB_DEPENDS+=	libc++.so:${PORTSDIR}/devel/libc++
LDFLAGS+=	-lc++
.endif
.endif
{% endhighlight %}

PR: 196707
