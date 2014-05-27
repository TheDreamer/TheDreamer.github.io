---
title: Fixed some annoyances I see in periodic emails
layout: post
comments: yes
category: patches
---

Every time I look at the various periodic emails from my FreeBSD systems, there are certain error messages that keep
appearing that are kind of annoying.  So, overtime I've made tweaks to some of the standard system periodic scripts to
address the issues.  Recently I thought, maybe I should see if they are PR worthy.

* `/etc/periodic/daily/100.clean-disks`

Every day this job is run, and every day it complains about .gvfs being inaccessable.

Linux counter used to have that annoying quirk, until I made a small change to the script to have it ignore such filesystem
types when doing `df`.  Now, it is not only set in cron to run once a week, but it also checks to see if there's a newer
version of the script before it runs.  But, because of this hack, I disabled the update.  Until CFEngine came along.  I
just have it check if the script needs updating, append `.new` to the filename and do this sometime before the script is
scheduled to run, rather than minutes before.

The policy file has something like this in it....

{% highlight cfengine3 %}
bundle agent ubuntu
{
vars:

    ubuntu::

    "lico_f"            string => "/home/$(g.lk)/bin/lico-update.sh";

    "lico_new_size"     string => "filestat("$(lico_f).new","size");

    "lico_min_size"     string => "30000"; #sanity check that it downloaded correctly

classes:

    ubuntu::

    "has_lico"          expression => fileexists("$(lico_f)");

files:

    ubuntu.has_lico::

    "$(lico_f)"

          comment       => "Update lico-update.sh and add gvfs exclude",
          perms         => mog("0755","$(g.lk)","$(g.lk)"),
          edit_defaults => empty,
          edit_line     => lico_df_ignore_fuse("$(lico_f).new"),
          ifvarclass    => and(
                    isnewerthan("$(lico_f).new","$(lico_f)"),
                    isgreaterthan("$(lico_new_size)","$(lico_min_size)")
                              );

}

bundle edit_line lico_df_ignore_fuse(f)
{
insert_lines:

          "$(f)"

                    insert_type => "file";

replace_patterns:

          "{DF} -l -P \|"

                    replace_with = AllWith("{DF} -l -P -x fuse.gvfs-fuse-daemon |");

}
{% endhighlight %}

Someday, I'll get around to adjusting bundlesequence so that agent bundle ubuntu or freebsd is only called for the
corresponding host type.

Where `$(g.lk)` is the username of my account.  I had switched to using a global variable, since I had envisioned the
possibility of using my home CFEngine to help manage my workstations at work.  Though not sure its going to ever happen.
I want there to be only one authoritative source of masterfiles (currently my subversion repository, but plans are to
convert to my own git repository.  Had set one up following "[Installing Git and Gitweb on FreeBSD _at unix-heaven.org_](1), which is the same site I used when getting start with CFEngine.

But, I broke it and haven't had time to see what's wrong.  Have also been looking to see about using a different web
interface, and using different servers to run on.  Perhaps doing replication and CARP.  Unless I find an alternative way of
trying HAST, had intended to leave a paritition for it, but seems I forgot.

I had heard of using git pull to maintain a policy server, but not planing to pay for my own private git repository for
this currently.  Though I wonder if there's a way to use Dropbox to do this? :flushed:

Anyways, the patch I made to `/etc/periodic/daily/100.clean-disks` is to have add `-fstype fusefs` to the list of
file-system types that it 'prunes' from `find`.

* `/etc/periodic/security/100.chksetuid`

This patch was to have it handle mounts where there are spaces in a directory name.  This is namly because I had made the
`VirtualBox VMs` directory in my home to be a separate ZFS dataset.  This was largely to experiment with different settings
to try to improve my VirtualBox performance with the harddisk images.  This was partly because I had turned on dedup for
`/home`, which was necessary because I had 3 or 4 copies of Zen (my Windows 7 desktop, before I turned into a FreeBSD system
named 'zen').  Some how when I setup the Windows computer, the hostname had become capitalized.  My Windows VMs continue
to have capitalized names...guess that's they way it went.  Guess I never paid attention to it.  Or perhaps it was because
the Windows XP computer before Zen had an all uppercase hostname.  I had named her __`TARDIS`__ :grinning:

So, Zen had a pair of 1.5TB drives, which is now the mirrored zpool that mainly contains /home for zen.  Each salvage
attempt, resulted in about 700GB of data.  Which I was storing in my home directory, while I searched for some tool to 
efficiently diff these directories to easily keep the common files and examine the files that differed, etc.

The closet was meld, which after waiting 4 days showed a window.  Except it had used the settings I had from the last time
I used meld, which were not the settings I wanted for this comparison.  Ummm, no not going to wait another 4 days.

Perhaps some other date in the future...

At least I didn't have to worry about the contents of "My Documents", "My Music", "My Pictures", or "My Videos", since I
was syncing these directories to `orac` as an additional part of my backup strategy.   Its the applications that stored
data elsewhere that I'm trying to track down.  For example, I have recovered my Intellipap data...though I discovered
recently that the machine only saves about 3 months of detailed data...since I went for a year before doing a download
from my machine.  Wonder if I'm too compliant on CPAP now.

[1](http://unix-heaven.org/node/43)
