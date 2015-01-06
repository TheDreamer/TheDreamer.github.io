---
title: Update p5-IO-Interface on my nagios server broke DHCP?
layout: post
comments: yes
category: patches
---

Well, its causing nagios to report that my DHCP servers are critical.  Though the actual problem is `"Illegal Seek"`.

But, where is the problem?  After some searching around, I discover `Devel::Trace`.  This allows me to narrow down to where
the error message is being emitted.  Searching around to see what perl Module(s) that I recently updated:

p5-Exporter-Tiny-0.042_1 installed
p5-List-MoreUtils upgraded: 0.33_1 -> 0.402 
p5-namespace-autoclean upgraded: 0.20_1 -> 0.23 
p5-Path-Class upgraded: 0.34_1 -> 0.35 
p5-IO-Socket-IP upgraded: 0.34 -> 0.35 
p5-IO-Interface upgraded: 1.06_2 -> 1.09 

I eventually narrow in on `net/p5-IO-Interface`.

The method being used in `check_dhcp.pl` is the _deprecated_ method, so I whip up a quick patch to use the _preferred_
way, and it seems to work.  So, tweak it so that it can go through `portshaker` and `poudriere testport`, and submit it as
a bug.  Then move it into my local injection point so that I can install it from my private pkg repository.  Kind of annoying
that testport doesn't leave the final port's pkg for use.  Though not sure how to move the pkg into a privately signed
repo, etc.

And, it'll take while because I had done an portsnap update earlier this evening to try to get a complete build of pkgs for
'zen', so that I can complete its upgrade to 9.3.  104 packages queued for my servers.  I'm sure it'll want to do 600+ for
'zen'.  No idea on what I'm doing to do about the FreeBSD servers at work yet.

Anyhoo....

PR: 196528
