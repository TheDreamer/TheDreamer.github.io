---
title: My Forgotten BackupPC patch
layout: post
comments: yes
category: patches
---

Somewhere in December/January, not sure if it was before the ports freeze at work.<sup>1</sup>  Because the patch had made
its way into my BackupPC instance at work.  But, it appears that in Perl 5.18, the 'random' order of hash keys was taken a
step further and now there is no guarantee that the 'random' order of the keys will be constant and portable.

This problem manifests in that the host config 'tabs' are managed using a hash, so the ordering of 'tabs' which had been
the same (non-alphabetical) order for as long as I've been running BackupPC (with Perl 5.8 -> 5.16) had been the same
everywhere.  This also includes different computers and operating systems (Ubuntu, Solaris, and FreeBSD.)  But, when
FreeBSD changed the default version of Perl from 5.16 to 5.18, I opted to make the leap to 5.20 *(which also introduced its
own set of annoyances.)*  I had previously lept to 5.16 from 5.12....

So now not only are the 'tabs' in a different order than before, but the order changes every time the page is refreshed.
And, that includes page refreshes from clicking on a different 'tab'.

The simple solution that I went with was to replace 'keys(%hash)' with 'sort(keys(%hash))'.  While this is a different order
than what I had previously grown accustom to, in time I will probably come to adjust and expect this new ordering.

Don't recall where I left the work on this patch for possible submission, only thought about it as I was reading what had
changed in the recent upgrade to BackupPC 3.3.  And, noting that I thought I was injecting a patch for one of the
deprecation 'annoyances'. (so such and such is deprecated now, but does it have to break or interfere with the application
for users?)  Anyways, couldn't find any evidence of where my version of that patch is (most likely, somebody had beat me to
the punch before I had gotten to where I would get close to submitting.)  But, wondered how it was that I was handling the
hash key order change.

I make tweaks to the `sysutils/backuppc` port (among other things) in one of two ways, using `edit_line` promises in
CFEngine or patches that I inject using either CFEngine (using `copy_file` and/or `edit_line`) or `ports-mgmt/portshaker`
into my private repo build process.<sup>2</sup>

There is one tweak that I currently do through a CFEngine `edit_line` promise that I have thought about converting into a
patch to possibly even submit upstream.  Though 4.0 was supposed to be imminent for years now... hmmm, alpha0 came out on
2013-06-23, the last is alpha3 on 2013-12-01.  With 3.3.1 being a surprise release....

-----

**1**: _Frozen at the commit just before sudden obliteration of gnome2 with the entrance of gnome3.  Didn't matter that it
was weeks, of once or twice daily build attempts, before I (and it seems lots of other people) could get gnome3 to build to
even attempt making the switch to it...at home.)  I do plan to someday soon to upgrade, as the freeze is causing problems
for other ports that I should be keeping current._

**2**: _CFEngine driven 'portshaker' and 'poudriere' -- Which was already a complicated process/policy when it was a
CFEngine supported, cron driven process.  So, not sure if/when I might contemplate sanitizing it for sharing with the world.
Not to mention that there are other CFEngine snipets that I've thought about sharing, including a post or two that I had
started over on my old [blog](http://lawrencechen.net)._
