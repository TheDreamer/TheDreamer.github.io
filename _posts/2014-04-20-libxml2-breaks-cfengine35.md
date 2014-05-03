---
title: libxml2 update breaks my cfengine
description: "Due to the library version of libxml2 being brought in line with what upstream expects, my cfengine installations broke.  Even though I did reinstall systuils/cfengine35 as part of this work.  Problem is cf-execd is trying to run cf-agent from /var/cfengine/bin, so it can't update itself or do anything."
layout: post
comments: yes
category: patches
---

Due to this entry in `/usr/ports/UPDATING`

```
20140416:
  AFFECTS: users of print/freetype2 textproc/libxml2 x11/pixman
           x11/libxcb and graphics/freeglut
  AUTHOR: x11@FreeBSD.org and gnome@FreeBSD.org
                 
  The library version of the above libraries has been brought in line with what upstream expects.
```

my cfengine35 installations broke.  Even though I did reinstall cfengine35 as
part of this work.

Tried portmaster, but ran into the "Argument list too long" error. So, switched
to pkg_libchk method.

Problem is cf-execd is trying to run cf-agent from /var/cfengine/bin, so it
can't update ifself or do anything.  Which I realized why previous co-worker
had cursed lots in getting cfengine22 to build fully static.

So, as I was working on trying to do this to cfengine35, I discovered the
problem reported as PR: [ports/180896][1] isn't the auto-activation of libxml2
in upstream configure getting it wrong, but that we don't have a knob to
correctly control this behavior.

```
LIBXML2_CONFIGURE_ON= --with-libxml2=${LOCALBASE}
LIBXML2_CONFIGURE_OFF= --without-libxml2
+
LIBXML2_LIB_DEPENDS= ...
```

Meanwhile, I got a patch in to either make cf-agent & cf-promises as fully
static executables, or partially static.  Where partially static, links all
*libtool* libraries as static (namely libpromises and libxml2.)  I had wanted
tokyocabinet as part of a partially static build, but there's no libtool
definition for this.

I went with partially static for my FreeBSD servers.

[1]: http://www.freebsd.org/cgi/query-pr.cgi?pr=ports/180896 "sysutils/cfengine35: can't use edit_xml because libxml2 not found"
