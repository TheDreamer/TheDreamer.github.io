---
title: "Cannot determine local time zone"
layout: post
comments: yes
category: patches
---

Some time after `DateTime::TimeZone` 1.71, my irssi restarted.  And, some time after that, I noticed that twirssi wasn't
running for some reason.  When I tried to get twirssi to load, it reported an error of "Cannot determine local time zone".
I eventually found that this is this is from a call to _die_ in `DateTime::TimeZone::Local`, and that through a variety
of troubleshooting measures that it only happened inside of irssi and that rolling back to `DateTime::TimeZone` 1.69 got
things working again.

At first I opened a [ticket](https://rt.cpan.org/Public/Bug/Display.html?id=97227) against `DateTime::TimeZone`, on July
14th, which quickly stalled on July 16th.

During this time, I had identified that the problem seemed to be with `DateTime::TimeZone::OlsonDB::Observance.pm`, where
replacing the 1.71 version with file from 1.69 also solved things.

Later I would see that a couple of plugins that I had written that use this module, were also dying.  At one point I
spotted the message _"Attempt to reload DateTime::TimeZone::America::Chicago.pm aborted"_, which threw me thinking there was
some kind of reentrancy problem.  Instead the problem was, that it had failed on an earlier attempt and that attempt was
cached in memory.

Had missed that at some point `DateTime::TimeZone` had updated to 1.73, followed by a restart of irssi...  and that the
plugins had stopped working.  So this past week, some people started **highlighting** that my two plugins that use this
Perl module weren't working... so I finally decided to dive in to figure out what was wrong.  Eventually found that I could
delete the failed load attempt from the `%INC` hash, and then found when I dumped the hash that there were numerous other
failed loads.

Slowly working up through the dependencies, I found that List::AllUtils was emitting death due to List::Util v1.25 being
less than the desired 1.31 minimum.  The code wants the 'any' method exposed, but I don't see it being used anywhere.
List::AllUtils being a wrapper to expose methods first from List::Util and then List::MoreUtils, though the any method
was later added to List::Util in v1.33.

Noting that List::Util is part of base and is 1.25 due to my systems running Perl 5.16.  While I see that 5.16 is already
considered EOL, with 5.18 and 5.20 being the two stable releases, and development going into 5.21...  I suspect 5.16 went
EOL when 5.20 was released on May 27, 2014.

But, when I had originally installed my systems, the default Perl version was 5.12...and later when the default version
was changed to 5.14 (on June 30, 2012), I opted to jump my systems to 5.16.  5.16 became the default version on October 23,
2013.  I suppose some day soon (perhaps when I finally get my poudriere setup working), I'll be making the jump to 5.20.

In the meantime, the List::Util version problem is addressed by installing `lang/p5-Scalar-List-Utils` which the
`devel/p5-List-AllUtils` port correctly depends on when Perl version is less than 5.20.

So, why wasn't it working under irssi?  Well, its altering the `@INC` path by claiming the need to move the directory
where Irssi.pm is installed up in the search order.  And, its looking for the path INSTALLARCHLIB, which is where only
base Perl arch specific modules go.  It is _properly_ being installed under INSTALLSITEARCH, where normally the 'site_perl'
directories are searched before the base ones.

With irssi forcing base arch to come before the other paths, its finding the base List::Util first.

I then found that I could override what path it was promoting by setting this in the config file:

{% highlight text %}
settings = {
    "perl/core" = {
        perl_use_lib = "/usr/local/lib/perl5/site_perl/5.16/mach";
    };
};
{% endhighlight %}

or

{% highlight text %}
/set perl_use_lib /usr/local/lib/perl5/site_perl/5.16/mach
/save
{% endhighlight %}
_and restarting irssi._

Not sure if it would've been possible to set the value to nothing, suspect that just makes it go back to the compiled in
value.

I later created a patch, for possible submission as a FreeBSD bug/patch.  But, opted to open an
[issue](https://github.com/irssi/irssi/issues/132) against irssi.
