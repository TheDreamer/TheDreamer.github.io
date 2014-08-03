---
title: "ports-mgmt/pkg (1.3.4) quiet delete doesn't"
layout: post
comments: yes
category: patches
---

Decided that it was time to cleanup the pkg's installed on my system.  Since the other day I watched it take forever to
update a port, that turned out to be a leaf package.  Leftover dependency from something I choose not to keep around.  There
are also other big left over build dependencies that for whatever reason weren't marked as automatic or removed
automaticallly at the end of a portmaster (w/ --delete-build-only) run.

I ran `ports-mgmt/pkg_rmleaves`, having coming up an [ugly fix]({% post_url 2014-02-19-pkg-rmleaves-patch %}) to finally
make it work for me... which later resulted in the `pkg_rmleaves-20140222` version.

Well, I selected a bunch of packages, and then it disappeared for a long time to seemingly delete the packages.  It locked
the package db, because a while later while it was still hiding I had typed 'pkg version -vIl\<' in another window to see
if the list of ports I need to update had gotten smaller.

Eventually it returned, and reported that there were no more leaves.  Which was odd, since there should be many packages
starting with `fpc-` still to be removed.  Suspect its a problem in the post 1.3.0 release of `ports-mgmt/pkg`.  Where
1.3.1 was especially fun where it segv'd on install and left me without any `pkg`, and re-bootstrapping it pulls in the
1.2.7 version where it'll complain lots that the database is much newer than it expects.

There was a portmaster backup of the old package, but I couldn't install it with a working `pkg`.  Eventually, I restored
`pkg-static` from backup....I had broken all 3 of my home FreeBSD systems.  And, then locked it so that I could still update other ports without it trying to update to broken release.

A few hours later, 1.3.1_1 was released....before I got to it, 1.3.2 came out.  Followed by an update to 1.3.3 and today
the update to 1.3.4.

Looking at what pkg_rmleaves was doing.  `pkg delete -q -y "$p"`  I narrowed the problem to the '-q' switch.

And, while I was filling out the bug report, I peeked at the source code (since I don't have portmaster clean after an
update on `zen`.) and found this:

{% highlight c %}
if (!quiet || dry_run) {
        print_jobs_summary(jobs,
            "Deinstallation has been requested for the following %d packages "
            "(of %d packages in the universe):\n\n", nbactions,
            pkg_jobs_total(jobs));
        if (dry_run) {
                retcode = EX_OK;
                goto cleanup;
        }
        rc = query_yesno(false,
                    "\nProceed with deinstalling packages [y/N]: ");
}
if (!rc || (retcode = pkg_jobs_apply(jobs)) != EPKG_OK)
        goto cleanup;
{% endhighlight %}

`rc` is initialized as *false*, so if quiet the answer is always **no**!  That's not right.

Had `query_yesno()` been called....it understands the 'quiet' flag, with:

{% highlight c %}
/* We use default value of yes or default in case of quiet mode */
if (quiet)
        return (yes || r);
{% endhighlight %}

*Guess I'm not getting to that new thing I was going to try to get going this weekend....*

PR: 192356
