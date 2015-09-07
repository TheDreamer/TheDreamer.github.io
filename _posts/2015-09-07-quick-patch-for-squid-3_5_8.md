---
title: Quick patch to www/squid 3.5.8 for TP_IPF on FreeBSD 9.3
-date:
-description:
layout: post
comments: yes
category: patches
---

The port got upgraded to 3.5.8 (from 3.5.7), but it won't build with TP_IPF.  The next day 3.5.8_1 came out with a fix for
TP_IPF build.  Not sure if I had this update when it was first encountered by my automated *poudriere* setup.  But, it
failed again on the next run, so there's something else wrong.

Digging into the code found where the offending block of code was...and that it was to clutter up logs nagging that I need
to upgrade to FreeBSD 10.x (effectively...since FreeBSD 9.3 comes with IPF v4.1.28, while it was in 10 that IPF was
upgraded to v5.1.2.)

The whole annoying bit was part of what changed between 3.5.7 and 3.5.8.

{% highlight udiff linenos %}
diff -u -r -N squid-3.5.7/src/ip/Intercept.cc squid-3.5.8/src/ip/Intercept.cc
--- squid-3.5.7/src/ip/Intercept.cc	2015-07-31 23:08:17.000000000 -0700
+++ squid-3.5.8/src/ip/Intercept.cc	2015-09-01 12:52:00.000000000 -0700
@@ -200,6 +200,19 @@
     // all fields must be set to 0
     memset(&natLookup, 0, sizeof(natLookup));
     // for NAT lookup set local and remote IP:port's
+    if (newConn->remote.isIPv6()) {
+#if IPFILTER_VERSION < 5000003
+        // warn once every 10 at critical level, then push down a level each repeated event
+        static int warningLevel = DBG_CRITICAL;
+        debugs(89, warningLevel, "IPF (IPFilter v4) NAT does not support IPv6. Please upgrade to IPFilter v5.1");
+        warningLevel = ++warningLevel % 10;
+        return false;
+#else
+        natLookup.nl_v = 6;
+    } else {
+        natLookup.nl_v = 4;
+#endif
+    }
     natLookup.nl_inport = htons(newConn->local.port());
     newConn->local.getInAddr(natLookup.nl_inip);
     natLookup.nl_outport = htons(newConn->remote.port());
{% endhighlight %}

Don't know why it emits a compiler warning, which is treated as fatal by the build.  But, my fix was to just comment
out the nag.  Though I shouldn't have run into it anyways, since I don't yet have IPv6.  Someday I will, but how networking
should get me away from needing to build squid with this option.

I do see that for 3.5.8, squid.conf.documented changes the note that 'intercept' disables 'authentication and IPv6' on the
port, to just disables 'authentication'.  

Anyways, the quick fix is comment out the nag that is causing compile failure.  FreeBSD 9.3 has 4.1.28, while FreeBSD 10
got 5.1.2.  Guess nobody wanted to backport 5.1.2 to 9....

Already spent to much time on computer today...yet again.

PR: 202950
