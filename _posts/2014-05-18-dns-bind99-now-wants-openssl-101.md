---
title: "dns/bind99 now wants security/openssl, but I don't!"
description: "Today when I updated 'dns/bind99' to 9.9.5_16. Where, I found it suddenly wanted 'security/openssl'.  But, I don't.  Since this breaks everything else that is using base openssl.  So, I avoid ports or port options that want this."
layout: post
comments: yes
category: other
---

Today when I updated  `dns/bind99` to

    PORTVERSION=	9.9.5
    PORTREVISION=	16

Not sure why there are so many PORTREVISIONS, some reversing previous ones.  Seems contrary to what I expect for FreeBSD.
Guess its a good thing that we won't be replacing our DNS servers at work with FreeBSD now.

Well, with this _PORTREVISION_, it is wanting `security/openssl`.  But, I don't.  Since this breaks pretty much everything
else that is using base openssl.  So, I avoid ports or port options that pull this in.

Well, the commit message is to fix build with GOST (on 10, base OpenSSL doesn't provide it) and to use OpenSSL from ports
for <10s where base OpenSSL is 0.9.8y....  Except GOST is port option that is off by default.  And, I have no need for it.
The default port options are: `IPV6` `SSL` `THREADS`.  While I'm not currently doing IPv6, I leave these options set when
I build `dns/bind99`.  SSL is required for DNSSEC, which I'm doing both at home and at work, and both have been working
fine with openssl-0.9.8 (various letters, since default is rather low but later I switched to running bind in zones at
work, so it has the latest `'y'` version.)

Other port options that I have set on FreeBSD are: `DOCS`, `FILTER_AAAA`, `IDN`, `LARGE_FILE`, `NEWSTATS`, `REPLACE_BASE`,
`RRL`, and `SIGCHASE`.  With the exception of `REPLACE_BASE`, which is unique to pre-10 FreeBSD and `DOCS`, this matches
how I build bind at work.

It is the `FILTER_AAAA` option that prompted me to switch to ports bind for my systems at home.  Since there are some
braindead apps that insist on trying IPv6 addresses, even though its disabled on everything (except lo0).  Some where I've
even set the configuration options to disable it, it still tries.  Though someday I plan to set up IPv6 at home....so many
projects, so little time and too many distractions from newer projects.

PR: ports/189931
