---
title: Revisiting graphics/shotwell after webkit-gtk3
layout: post
comments: yes
category: patches
---

In finally getting around to examining the response to the PR I had submitted in previous [post][1], last weekend I had
eventually settled on a simplier patch for this problem.  One that avoids the messiness of using OSVERSION, without OPSYS
for places where FreeBSD ports are used by other distros without means to abstract these details away, and resolves what
the correct solution for the OPENMP option is.

{% highlight udiff linenos %}
--- Makefile.orig	2015-02-21 09:11:52.000000000 -0600
+++ Makefile	2015-03-22 10:21:38.110064518 -0500
@@ -35,17 +35,14 @@
 CONFIGURE_ARGS=	--prefix=${PREFIX} --disable-icon-update
 CONFIGURE_ENV+=	--define=NO_CAMERA
 INSTALLS_ICONS=	yes
+PORTSCOUT=	limitw:1,even
 
 OPTIONS_DEFINE=	OPENMP
 OPTIONS_DEFAULT=
 OPENMP_DESC=	libraw uses OpenMP (implies GCC 4.6+)
 
-.include <bsd.port.options.mk>
-
-.if ${PORT_OPTIONS:MOPENMP}
-USE_GCC=	yes
-LDFLAGS+=	-lc++
-.endif
+OPENMP_USES=		compiler:gcc-c++11-lib
+OPENMP_USES_OFF=	compiler:c++11-lib
 
 SHEBANG_FILES=	${WRKSRC}/${CONFIGURE_SCRIPT} ${WRKSRC}/chkver
{% endhighlight %}

Has resulted in this patch getting incorporated into a different PR, that upgrades to newer release, without back reference
to my PR ([196707][2]) and another PR ([196236][3]) 

PR: 198986

[1]: {% post_url 2015-01-14-7-graphics-shotwell-after-webkit-gtk3-update %}
[2]: https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=196707
[3]: https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=196236
