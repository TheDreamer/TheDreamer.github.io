---
title: fix build of net/p5-IO-Interface-1.09 on FreeBSD
layout: post
comments: yes
category: patches
---

After this port was upgraded from 1.06 to 1.09, I found my `net-mgmt/nagios-check_dhcp.pl` port had stopped working.  I
jumped the gun by assuming that the problem was due to this port, and likely to its use of IO::Interface in a manner that
had been marked deprecated.  However, the patch [PR#196528](https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=196528)
didn't solve this problem.

The problem that was being seen with `check_dhcp.PL` was that it would exit with:

{% highlight text %}
Error: Illegal seek
{% endhighlight %}

And, dmesg would fill up with lines like this:

{% highlight text %}
WARNING pid 10917 (perl): ioctl sign-extension ioctl ffffffffc0206933
WARNING pid 10917 (perl): ioctl sign-extension ioctl ffffffffc0206921
{% endhighlight %}

In deeper investigation, found that the problem was in `net/p5-IO-Interface`.

In looking at the change log, between 1.06 and 1.09.  The upstream applied a patch for a segfault in 1.07, and another in
1.08.  1.08 was also the first Git version.  In 1.09, converted to use Module::Build.

I waited to see if this port would get fixed, or rolled back to 1.08, but didn't seem like either wanted to take place. So,
I finally decided to dig into Module::Build a bit and figure out how to get it to work on FreeBSD.

Early on I had noticed that the `Makefile.PL` had a **CONFIGURE** sub, and that there was no equivalent in the `Build.PL`
file.  But, wasn't familiar with either methods.  But, since nobody seemed to be working on this problem, I decided that the
'pkg lock' on `p5-IO-Interface` was going to cause a problem at some point.  Perhaps when I get around to continuing the
upgrades of FreeBSD 9.1/9.2 systems to FreeBSD 9.3.

So I figured, out that in the old `Makefile.PL`, it tested it if was on FreeBSD, OpenBSD, or NetBSD and would set a define
of `-D__USE_BSD`.  It would then check to see if the `/usr/include/ifaddrs.h` was present, and, if so, define
`-DUSE_GETIFADDRS`.  And, then check if `/usr/include/net/if_dl.h` was present, for which it would define, when present,
`-DHAVE_SOCKADDR_DL_STRUCT`.

Then guessed that these should go into BUILD.PL as values for `'extra_compiler_flags'`.  First I fed this into my
local poudriere packages system so that I could test that `check_dhcp.pl` worked as expected.  Then I fed it
`'poudriere testport'`.  While waiting for that to complete, since at the very least the default Perl had changed since I
last did a *testport*, I went looking to see about how to submit a bug upstream -- [Bug #101985](https://rt.cpan.org/Public/Bug/Display.html?id=101985).

Then *poudriere*'s QA gave me a warning, which I then worked out how to address, and then another run to make sure
everything was right.  Then unlock the package, intentionally break things by upgrading to 1.09, in part to generate lines
in dmesg for posting, and then upgrade to my patch.  And, then its off to submit.

PR: 197404
