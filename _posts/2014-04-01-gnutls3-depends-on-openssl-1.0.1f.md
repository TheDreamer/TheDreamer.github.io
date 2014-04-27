---
title: "security/gnutls3 depends on security/openssl - v1.0.1f"
layout: post
comments: yes
category: patches
---

Back on March 5, 2014, this port was updated to v3.1.22 to address a couple of
security vulnerabilities.  Where it had a new dependency for openssl-1.x, which
was 1.0.1f at the time.

But, I didn't want that.  So, I tracked down the port that had installed this
port and removed it, along with other leaf packages.

But, word is that this port might be replacing gnutls, which would be a much
bigger mess to purge.

So, I went looking to see why gnutls3 wanted openssl-1.x.  And, made it an
option, preferring to build without it.

Its being pulled in by `dns/unbound`, which with its default build options
makes it require openssl from ports on <10 (since 10's base openssl is 1.0.1f.)
And, this is to create libgnutls-dane, which adds DNSSEC verification support
to DANE.  And, is needed for the `--check` option with danetool3.

Since I suspect for the majority of users this is just a library dependency to
them, I made the default to build gnutls3 without needing openssl-1.x.

PR: ports/188184

------

The dependency was added shortly after the previous 3.1.18 version had been
released, but there was no bump to the PORTREVISION, so it went unnoticed for
me.

I forget now what I had uninstalled when I nuked this port back on March 5th,
so not sure if I need this port on my system at the moment....
