---
title: patch archivers/ruby-lha for ruby 2.0
layout: post
comments: yes
category: patches
---

In UPDATING on 20141008, the default ruby version was updated from 1.9 to 2.0.  Long ago, in addition to listing where my
"default versions" differed from *default* (ie `mysql=55p` instead of `mysql=55`), I had added in definitions for all of the
then *default* versions to `DEFAULT_VERSIONS=` in my `make.conf` file.  Partly to avoid *surprise* changes, or from
forgetting to check `UPDATING` until after I had started updating my ports.  With the added benefit that if I'm not ready to
make the switch, I don't have to right away.  Additionally, now that I'm doing unattended daily poudriere builds, it
avoids surprises to my package repo (to some extent....)

With poudriere, I have two repos being built.  One for my *headless 'servers'* and one for *'zen'*.  For the first case,
I was able with minor adjustments to get all my packages to build and have converted to updating those two systems from
that repo.  Getting the repo for *'zen'* continues to be a work in progress.

At the time of the siwtch, the only use of `lang/ruby19` was by `ports-mgmt/portupgrade`, so I deleted `portupgrade` and
the resulting leaf packages from these systems.  However, later I found that I still needed parts of it for other
automated tasks (`portsclean`) and the desire/need to occasionally build a port on these systems (by a means other than
with `portmaster`, my current preferred tool...though there are things I miss that `portupgrade` could do.)  So, I changed
these systems' repo to the new default, and (re-)installed 'portupgrade' from it.

Since the repor for *'zen'* was still a work in progress, I decided to be risky and switch its default to eventually make
that transition.  The next day after making the switch, I discovered that this port to be one that didn't survive the
switch to 2.0.

I found a post 0.8.1 commit to github that addressed the **BROKEN** issue, and threw in a patch to silence the deprecated
function warning (which is removed from CAPI in 2.2.) and the #include warning.

As I was about to unleash this patch, I discovered that between Nov 1st and Nov 21st. The port had been updated to show
that it also doesn't build with ruby 2.1.  So, with a bit of work, I run `poudriere testport` with against both ruby 2.0
and ruby 2.1.

PR: 195268
