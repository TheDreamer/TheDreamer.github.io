---
title: Some WIP commits
layout: post
comments: yes
category: patches
---

After finally, breaking down and looking at what patches to get `ports-mgmt/portrac` version 0.5 to at least compile.  I
looked at the current state of uncommitted patches that I've been working on, and saw that I had been in the middle of
making a *`git commit`* on my patches to `math/orpie`.  I couldn't recall if things we working yet, because I'm still running
the prior version.  And, currently poudriere is rebuilding everything from scratch, due to having done `freebsd-update` to
the jails recently (though what was patched shouldn't have affected anything?)  Plus there not being a kernel bump, I had
to do my extra 'poudriere jail -U' hack to make the push back **`UNAME_r`** and **`UNAME_v`**.  So, that the `systuils/lsof`
produced by poudriere won't complain about kernel version mismatches.

Since I'm waiting for the rebuild, which has gotten interrupted a few times, to complete.  I haven't had a chance to see
if my changes work.  Had thought about creating a new parallel jail so that I could attempt single build bulk runs while the
full package list bulk run is in progress, but in trying to decide how to avoid duplication of effort and packages, and
reintegration into my package repository the whole thing never got further than a few changes that I had done to my
'poudriere' CFEngine policy.  All those changes vanished with a quick `svn revert`.  Someday I'll get around to converting
my CFEngine revision control to *`git`*. (the project would go faster if somebody else has published that they already have
a *`git`* setup on **CARP/HAST**...on FreeBSD 9.3 :wink:)

But, there's a lot of uncommitted work-in-progress fiddling to ports that I should at least commit so that its saved
somewhere.  Especially, since while I was away at the Mayo Clinic, my computer evidently locked up somewhere before the
`ichwd` gets restarted.  Not sure why/where, since I couldn't unblank the screen when I got back.  So, I power cycled the
box back up.

I had thought, in a pinch, I could figure out how to access some of my old medical records by remote (which I had done
during an earlier trip.)  Had also thought about working from remote a bit to get them into the instance of `www/owncloud`
to make things eaiser (at least for the duration.)  Should also try and figure out how to get notifications from my nagios
working again, namely because I've switched smartphones and the imap account (hosted at 1&1) hasn't make the move.  It
doesn't seem to work the same way, perhaps due to how `mail/fetchmail` affects flags on messages left on server.  Have
meant to look at `mail/getmail`, which has been suggested as the solution.  My iCloud account was also impacted by my use
of `mail/fetchmail`, which I resorted to disabling that fetchmail user.  (I use different users to separate providers,
whether messages are kept, different intervals, the use of IDLE, or other special handling/processing, resulting in
currently 13 [was 14] fetchmails.)

Someday I'll also figure out a better way to handle the flood of e-mail (aside from the current mainly ignore, and the
unpredictable random check of select accounts; or when an out-of-band signal prompts attention.)

Probably time to resurrect 'orac', where I used to mirror my *Documents* folder on.  And, perhaps time to look at some
cloud based solution.  Or take the plunge and finish getting things into **Healthvault**, something I had started when it
was **Google Health**....but mainly use for getting results from LabCorp, or uploads of from my WiThings scale (which I
should start using again, now that I've lost weight...) or my Omron 10 Plus (wonder if I want to upgrade to the new Wireless version, supports both Android and iOS...though I try to keep such things away from bedroom...for sleep hygiene.)

*Upgrading to the wireless version might become necessary if I make good on my threat to eliminate Windows completely...*

------

2014-06-14: Apparently, I also missed that I had comments about WIP on updating my patches to dovecot2... currently,
indexing is busted even with current patches, will probably need to inject some debug code.  This effort being delayed due
to oversight on the impact of updating my jails...
