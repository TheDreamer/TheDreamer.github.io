---
title: "auto-nice daemon vs. CFEngine"
layout: post
comments: yes
category: other
---

Recently, my FreeBSD system at work was being crippled due to certain processes running at normal priority that weren't
playing nice and weren't really that high of priority compared to what I needed to get done on the system.

While that describes more than just the specific process at issue, the distinction is this is a long running process.

Namely, cf-agent is kind of a hog when its running, but generally its only around for a minute or two (though its been
inching to >5 minutes... >10 minutes on some systems, which is causing other problems....)  But, the recent problem process
is bpbkar, all 10 of them.  I had tried setting it to not run so many parallel jobs against the zpool, but the administrator
reset it because he wanted things to go as fast a possible.  And, while those backups have been useful in the past it is not the primary backup of my system.  I have a local backup that is for when I oop something, while the NetBackup is my system
won't boot because importing the zpool panics the boot... which seemed to come down to [Intel's Cougar Point SATA Bug][1]
plus other components.  With the drives now running off a SiI3132 card and things got a stable.  It was somewhat annoying
that a mid-2012 purchased Dell Optiplex 990, would be using the faulty chipset (*B2 Stepping*) reported in Jan 2011.

It took a lot of convincing to get Dell to replace the motherboard with one using corrected chipset (*B3 Stepping*), after
which it complained about one of the DIMMs (apparently it was marginal in some way....)  But, I continue to run my pair
of drives from the SiI3132 card.  Though sometimes wonder if they would get any benefit from the 6-Gbps SATA ports.  I had
had a heck of time finding drives that weren't DOA or die early, eventually convinced purchasing to get drives from a 
different supplier.  Reinforcing my personal ban on buying drives from a certain vendor.  Plus these days, Amazon Prime
fits my needs much better :wink:

At first they were all post-Thailand Seagate drives with only 1-year warranties.  I tried pushing for ones with longer,
hoping for better quality.  Got a pair with 2 year warranties....similar to what I was using at home.  Well, the home
pair have both died...within a month of each other, even though one was purchased ~3 months earlier (and was extracted from
external enclosure...it was also my first encounter with trying to replace a drive in an array built using 512-byte sector
drives with one that is of Advanced Format.  Eventually, I got a second and made a new mirror to move things to....)  I
didn't get around to purchasing a replacement before the second drive failed....  Some of the data is living exclusively
in backups at the moment.

The drives failed pretty much the same way at work.  Seems Seagate had discontinued the model a quarter or two before I had
received the drives.  Leaving only 1-year warranty drives (in the consumer space).  Anyhoo, they have both since been
replaced with WD Purple drives.  Which I'm finding aren't that great, apparently they're so write optimized that reads
suffer.  I guess in video surveillance is write fast and hard, and hope to never have to read things back, etc.  But, at
the time it was hard to tell what exactly the different between Red and Purple were, with Purple sounding better than Red.
And, not just because its "Purple" :smiling_imp: !  Plus at the time 2TB Purple drives were cheaper than 2TB Red drives,
but this was vendor dependent.  The drives ended up coming from a vendor where this wasn't the case.

But, better to learn what Purple is like at work than at home :innocent:

So, another reason why 10 parallel bpbkar's was impacting system performance more is that the change from our old NetBackup
system to the new NetBackup, was that the zpool was also resilvering.  Initial estimates were getting long and longer with
bpbkar being so greedy, though I later boosted resilvering priority to get it complete sooner.  But, while this was
happening, I got to wondering if I couldn't run `and` to address this problem.

I already have `and` deployed on our public Unix (Solaris) servers (at work)....to nice down long running (12+ hours) matlab,
sas, or mathematica jobs....  So, I worked on trying to build `and` on FreeBSD.  But, while I was doing that, in reading
documentation, I found that it wasn't what I was wanting.  `and` does not touch root processes.  And, all the processes that
I want to have their priority managed are running as root.

So, I looked at CFEngine.

First, I had decided what I wanted to do for these high demanding processes was to move them to idle priority.  So, I came
up with a dirty little script named **mkprocidle**

{% highlight sh %}
#!/bin/sh
if [ $# -lt 2 ]; then
        echo "Usage: $0 <idprio> <procname> [full]"
        exit 1
fi
idprio=$1
procname=$2
pgrep_args=
[ -n "$3" ] && pgrep_args="-f"
for pid in `pgrep ${pgrep_args} ${procname}`; do
        state=`idprio ${pid} 2>&1 | grep normal`
        if [ $? -eq 0 ]; then
                idprio ${idprio} -${pid}
        fi
done
{% endhighlight %}

And, then to come up with some promises in CFEngine:

{% highlight cfengine3 linenos %}
bundle agent proc_man
{
vars:

    procman::

        "mkprocidle"    string => "$(g.workdir)/bin/mkprocidle";

    procman.freebsd::

        "compilers"     slist => { "cc1", "cc1plus",
                                   "clang", escape("clang++:"),
                                   "gcc, escape("g++") };

processes:

    procman.mew::

        "bpbkar"

                    comment => "Move NetBackup Jobs to idle class (i10)",
             process_select => process_priority("20","100"),
               process_stop => "$(mkprocidle) 10 bpbkar";

    procman.freebsd::

        "$(compilers)"

                    comment => "Reduce impact of updating ports",
             process_select => process_priority("20","100"),
               process_stop => "$(mkprocidle) 2 $(compilers)";

        "BackupPC_dump"

                    comment => "Move BackupPC dumps to idle class (i1)",
             process_select => process_upriority("backuppc","20","100"),
               process_stop => "$(mkprocidle) 1 BackupPC_dump full";

}
{% endhighlight %}

*process_priority* and *process_upriority* are defined as:

{% highlight cfengine3 %}
body process_select process_priority(l,u)
{
priority => irange("$(l)","$(u)");
process_result => "priority";
}

body process_select process_upriority(user,l,u)
{
process_owner => { "$(user)" };
priority => irange("$(l)","$(u)");
process_result => "process_owner.priority";
}
{% endhighlight %}

In my library file, I have this note with these bodies.

    # FreeBSD
    # -------
    # process priorities are 0-255 (but ps subtracts 100)
    #
    # Interrupt Threads     =   0 to  47 = -100 to -53
    # Real-Time Threads     =  48 to  79 =  -52 to -21
    # top half kern threads =  80 to 119 =  -20 to  19
    # normal user threads   = 120 to 223 =   20 to 123
    # idle priority threads = 224 to 255 =  124 to 155
    #
    # nice is just a hint...threads set to idle class can have any value for NI,
    # including 0.
    # rtprio field would be nice, but don't feel like teaching CFEngine how to
    # parse it.
    # So, make CFEngine only get PRI on FreeBSD (since it tries NI before PRI)
    #
    # linux is different
    # nice 0 is 19...max nice is 0, very unnice is -20 is 39.
    # 40+ is kernel/realtimme stuff...no idle class
    # CFEngine only looks at "NI".

So these are my patches to `sysutils/cfengine35` to make this work:

**patch-libpromises-classes.c**
{% highlight diff linenos %}
--- libpromises/classes.c.orig  2013-12-09 06:13:14.000000000 -0600
+++ libpromises/classes.c   2014-06-26 13:34:36.196280521 -0500
@@ -76,7 +76,7 @@
     "-N -eo user,pid,ppid,pgid,pcpu,pmem,vsz,ni,stat,st=STIME,time,args",  /* aix */
     "-eo user,pid,ppid,pgid,pcpu,pmem,vsz,ni,rss,nlwp,stime,time,args",        /* linux */
     "-eo user,pid,ppid,pgid,pcpu,pmem,vsz,pri,rss,nlwp,stime,time,args",        /* solaris */
-    "auxw",                     /* freebsd */
+    "axwwo user,pid,ppid,pgid,pcpu,pmem,vsz,pri,rss,nlwp,start,time,args", /* freebsd */
     "auxw",                     /* netbsd */
     "-elyf",                    /* cray */
     "-aW",                      /* NT */
{% endhighlight %}

**patch-libpromises-mod_process.c**
{% highlight diff linenos %}
--- libpromises/mod_process.c.orig  2013-12-09 06:13:14.000000000 -0600
+++ libpromises/mod_process.c   2014-06-26 17:25:04.786735872 -0500
@@ -42,7 +42,11 @@
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

Should probably think about pushing this upstream someday....








[1]: http://www.anandtech.com/show/4143/the-source-of-intels-cougar-point-sata-bug
