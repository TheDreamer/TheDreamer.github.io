---
title: Getting closer to pushing FreeBSD process priorites up into CFEngine
layout: post
comments: yes
category: patches
---

In [auto-nice daemon vs. CFEngine]({% post_url 2014-06-27-auto-nice-daemon-vs-cfengine %}), I made had made a patch to
`sysutils/cfengine35` to enable finer grain process priority control on my FreeBSD servers.  Today, I decided that I
should see about getting it into the upstream, since 3.6 had already been released and more recently 3.6.1 has appeared.
Given the major problems I had when I accidentally upgraded myself from 3.4.4 to 3.5.0, because I had been using
`sysutils/cfengine`...  I had decided a while back to switch to using `sysutils/cfengine35` so that I can make the
leap to 3.6 on my own timeline (which could be never, though probably not since there's some interesting new things in it
that I will want to try eventually :pensive:)  Though I still have a lot of other things I want/need to do before I get
there.

So, I forked the current 'GH: *cfengine/core*', and looked at making my little patch.  Things are a little different now.

{% highlight diff %}
diff --git a/libpromises/systype.c b/libpromises/systype.c
index 5c5c95a..10c49e4 100644
--- a/libpromises/systype.c
+++ b/libpromises/systype.c
@@ -94,7 +94,7 @@ const char *const VPSOPTS[] =
     [PLATFORM_CONTEXT_LINUX] = "-eo user,pid,ppid,pgid,pcpu,pmem,vsz,ni,rss:9,nlwp,stime,time,args",        /* linux */
     [PLATFORM_CONTEXT_SOLARIS] = "auxww",     /* solaris >= 11 */
     [PLATFORM_CONTEXT_SUN_SOLARIS] = "auxww", /* solaris < 11 */
-    [PLATFORM_CONTEXT_FREEBSD] = "auxw",              /* freebsd */
+    [PLATFORM_CONTEXT_FREEBSD] = "axwwo user,pid,ppid,pgid,pcpu,pmem,vsz,pri,rss,nlwp,start,time,args",  /* freebsd */
     [PLATFORM_CONTEXT_NETBSD] = "-axo user,pid,ppid,pgid,pcpu,pmem,vsz,ni,rss,nlwp,start,time,args",   /* netbsd */
     [PLATFORM_CONTEXT_CRAYOS] = "-elyf",              /* cray */
     [PLATFORM_CONTEXT_WINDOWS_NT] = "-aW",            /* NT */
{% endhighlight %}

Interesting that *NetBSD* has its args for `/bin/ps` changed since 3.5.3, and not that different than what I had come up
for FreeBSD. :smirk:

{% highlight diff %}
diff --git a/libpromises/mod_process.c b/libpromises/mod_process.c
index c103884..279e823 100644
--- a/libpromises/mod_process.c
+++ b/libpromises/mod_process.c
@@ -42,7 +42,11 @@ static const ConstraintSyntax process_select_constraints[] =
     ConstraintSyntaxNewIntRange("pid", CF_VALRANGE, "Range of integers matching the process id of a process", SYNTAX_STATUS_NORMAL),
     ConstraintSyntaxNewIntRange("pgid", CF_VALRANGE, "Range of integers matching the parent group id of a process", SYNTAX_STATUS_NORMAL),
     ConstraintSyntaxNewIntRange("ppid", CF_VALRANGE, "Range of integers matching the parent process id of a process", SYNTAX_STATUS_NORMAL),
+#if defined (__FreeBSD__)
+    ConstraintSyntaxNewIntRange("priority", "-100,+155", "Range of integers matching the priority field (PRI/NI) of a process", SYNTAX_STATUS_NORMAL),
+#else
     ConstraintSyntaxNewIntRange("priority", "-20,+20", "Range of integers matching the priority field (PRI/NI) of a process", SYNTAX_STATUS_NORMAL),
+#endif
     ConstraintSyntaxNewStringList("process_owner", "", "List of regexes matching the user of a process", SYNTAX_STATUS_NORMAL),
     ConstraintSyntaxNewString("process_result",
      "[(process_owner|pid|ppid||pgid|rsize|vsize|status|command|ttime|stime|tty|priority|threads)[|&!.]*]*",
{% endhighlight %}

Now the question is will I make a pull request? :pensive:
