---
title: Giving sysutils/pefs-kmod a try
description: "So, I decided to give PEFS a try.  After loading the module, I noticed a message in dmesg.  Later I wondered if my work machine had an i7 with AESNI.  So, I tried loading that module..."
layout: post
comments: yes
category: patches
---

So, I decided to give PEFS a try.

After loading the module, I noticed in dmesg:

    cryptosoft0: <software crypto> on motherboard

Later I wondered if my work machine had an i7 with AESNI, So, I tried loading
that module.

This message appeared in dmesg:

    aesni0: <AES-CBC,AES-XTS> on motherboard

But, then I noticed that AESNI support is disabled in the build....its turned
on by a DEFINE.  Probably to have it build on a wider range of FreeBSD versions
it was disabled.  though would be nice if it could be turned on for people
that want it and know they can, better if it could also tell if you can or
can't have it.

So, I quickly added a knob to turn it on.

Now when I load pefs on my work machine, I get:

    pefs: AESNI hardware acceleration enabled

in dmesg.

Now trying this at home 9where I had only built it, but never got around to
trying it), I first try loading aesni.

These two lines appeared in dmesg:

    cryptosoft0: <software crypto> on motherboard
    aesni0: No AESNI support.

I then uploaded aesni, resulting in this line in dmesg:

    cryptosoft0: detached

Rebuilt the port with AESNI enabled, and loaded it resulting in the same two
messages with loading aesni.  Guess that is expected, but wasn't known because
I did things out of order at work.

No message from pefs.

However, what I ideally want is something that'll work from between home and
work, and across multiple operating systems (at least Linux and FreeBSD.)  So,
not sure if this fulfills that need, though there is a use for it on work
computer.

PR: ports/187620

------

2014-05-05 - sysutils/pefs-kmod is updated to 2014.03.31, which strangely
contains what largely looks like this patch but with no reference to my PR.
The audit trail for my PR only has the one entry, showing that my PR/patch
went over to the maintainer on 2014.03.15.

The difference is my patch uses the current OptionsNG for a single line of

    AESNI_MAKE_ENV=   PEFS_AESNI=yes

The difference is my patch uses the current OptionsNG of AESNI_MAKE_ENV,
while the released update converts this single line into:

    .include <bsd.port.options.mk>

    .if ${PORT_OPTIONS:MAESNI}
    MAKE_ENV+=      PEFS_AESNI=yes
    .endif

Do I sound bitter?  Perhaps, but over which issue?  The lack of attribution
or that it got redone as non-OptionsNG.  :disappointed: 

Many of my previous PRs which were implemented the same way as existing
options, got committed by changing them to the OptionsNG way without
updating the existing options to it.  But, pretty much all of them
acknowledged my PR directly or indirectly.

While this port had no existing options, so no reason to go backwards ??? :confused:
