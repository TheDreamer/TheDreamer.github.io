---
title: "revisiting gmirror(8) says to use rc.early which is no longer available"
description: "All of a sudden, my FreeBSD system at work has been panic'ng.  But no crash dumps getting saved.  Evidently, the proposed fix through dumpon isn't sufficient.  Perhaps this quick patch to savecore is needed as well..."
layout: post
comments: yes
category: patches
---

All of a sudden, my FreeBSD system at work has been panic'ng.  But, no crash
dumps getting saved.

Evidently, the proposed fix through dumpon:

{% highlight udiff linenos %}
--- dumpon.orig	2013-10-27 16:15:05.000000000 -0500
+++ dumpon	2013-11-06 07:44:38.137819839 -0600
@@ -47,6 +47,13 @@
 		echo "No suitable dump device was found." 1>&2
 		return 1
 		;;
+	/dev/mirror/*)
+		mirror_balance=`gmirror list ${dumpdev#/dev/mirror/} | grep ^Balance:`
+		mirror_balance=${mirror_balance#Balance: }
+		gmirror configure -b prefer ${dumpdev#/dev/mirror/}
+		dumpon_try "${dumpdev}"
+		gmirror configure -b ${mirror_balance} ${dumpdev#/dev/mirror/}
+		;;
 	*)
 		dumpon_try "${dumpdev}"
 		;;
{% endhighlight %}

is not sufficent to full solve the issue.

Perhaps this additional patch I whipped up today will solve it:

{% highlight udiff linenos %}
--- savecore.orig	2014-04-29 14:01:01.927926553 -0500
+++ savecore	2014-04-29 14:01:18.011928417 -0500
@@ -57,6 +57,10 @@
 	[Aa][Uu][Tt][Oo])
 		dev=
 		;;
+	/dev/mirror/*)
+		mirror_balance=`gmirror list ${dumpdev#/dev/mirror/} | grep ^Balance:`
+		mirror_balance=${mirror_balance#Balance: }
+		gmirror configure -b prefer ${dumpdev#/dev/mirror/}
 	*)
 		dev="${dumpdev}"
 		;;
@@ -70,6 +74,10 @@
 	else
 		check_startmsgs && echo 'No core dumps found.'
 	fi
+
+	if [ -n "${mirror_balance}" ]; then
+		gmirror configure -b ${mirror_balance} ${dumpdev#/dev/mirror/}
+	fi
 }
 
 load_rc_config $name
{% endhighlight %}

Meanwhile, I recompiled all the kernel modules from ports to see if that's
the reason.

PR: docs/178818

Ref: [Previous Post]({% post_url 2013-05-21-gmirror-says-to-use-rc-early %})
