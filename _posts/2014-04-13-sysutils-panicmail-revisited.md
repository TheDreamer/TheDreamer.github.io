---
title: revisiting the hacking of sysutils/panicmail
layout: post
comments: yes
category: patches
---

After updating 9.2-RELEASE-p4, which has a kernel.symbols file.  Wonder why -p3
didn't.  And, not rebuilding a GENERIC debug kernel, decided I needed to fix the
logic for using kernel.symbols.

Decided to remove the kernel.symbols stuff from /usr/local/etc/rc.d/panicmail
and patch /usr/sbin/crashinfo to use kernel.symbols if detected or specified
on commandline with kgdb.

{% highlight udiff linenos %}
--- crashinfo.orig	2013-10-05 23:02:24.907006000 -0500
+++ crashinfo	2014-04-13 14:14:10.435929691 -0500
@@ -31,7 +31,7 @@
 
 usage()
 {
-	echo "usage: crashinfo [-d crashdir] [-n dumpnr] [-k kernel] [core]"
+	echo "usage: crashinfo [-d crashdir] [-n dumpnr] [-k kernel] [-s kernel.symbols] [core]"
 	exit 1
 }
 
@@ -67,8 +67,10 @@
 CRASHDIR=/var/crash
 DUMPNR=
 KERNEL=
+KERNELS=
+SYMBOLS=
 
-while getopts "d:n:k:" opt; do
+while getopts "d:n:k:s:" opt; do
 	case "$opt" in
 	d)
 		CRASHDIR=$OPTARG
@@ -79,6 +81,9 @@
 	k)
 		KERNEL=$OPTARG
 		;;
+	2)
+		KERNELS=$OPTARG
+		;;
 	\?)
 		usage
 		;;
@@ -145,17 +150,32 @@
 	echo "$KERNEL not found"
 	exit 1
 fi
+# If the user didn't specify a kernel.symbols, then see if it exists.
+# if not, set KERNELS to KERNEL for kgdb
+if [ -z "$KERNELS" ]; then
+	if [ -e $KERNEL.symbols ]; then
+		KERNELS=$KERNEL.symbols
+		SYMBOLS="-s $KERNELS"
+	else
+		KERNELS=$KERNEL
+	fi
+elif [ ! -e $KERNELS ]; then
+	echo "$KERNELS not found"
+	exit 1
+else
+	SYMBOLS="-s $KERNELS"
+fi
 
 echo "Writing crash summary to $FILE."
 
 umask 077
 
 # Simulate uname
-ostype=$(echo -e printf '"%s", ostype' | gdb -x /dev/stdin -batch $KERNEL)
-osrelease=$(echo -e printf '"%s", osrelease' | gdb -x /dev/stdin -batch $KERNEL)
-version=$(echo -e printf '"%s", version' | gdb -x /dev/stdin -batch $KERNEL | \
+ostype=$(echo -e printf '"%s", ostype' | gdb -x /dev/stdin -batch $SYMBOLS $KERNEL)
+osrelease=$(echo -e printf '"%s", osrelease' | gdb -x /dev/stdin -batch $SYMBOLS $KERNEL)
+version=$(echo -e printf '"%s", version' | gdb -x /dev/stdin -batch $SYMBOLS $KERNEL | \
     tr '\t\n' '  ')
-machine=$(echo -e printf '"%s", machine' | gdb -x /dev/stdin -batch $KERNEL)
+machine=$(echo -e printf '"%s", machine' | gdb -x /dev/stdin -batch $SYMBOLS $KERNEL)
 
 exec > $FILE 2>&1
 
@@ -174,7 +194,7 @@
 if [ $? -eq 0 ]; then
 	echo "bt" >> $file
 	echo "quit" >> $file
-	kgdb $KERNEL $VMCORE < $file
+	kgdb $KERNELS $VMCORE < $file
 	rm -f $file
 	echo
 fi
{% endhighlight %}
