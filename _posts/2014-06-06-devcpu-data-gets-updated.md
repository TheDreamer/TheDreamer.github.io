---
title: sysutils/devcpu-data get's updated...sort of
layout: post
comments: yes
category: patches
---
I've been working on my own update to this port for sometime, where the stumbling block has been trying to locate a tool
to decode AMD microcode files.  The attempt to resurrect [devel/ruby-mmap]({% post_url 2013-11-19-ruby-mmap %}) was related
to the attempt.  At that time, I was trying to update the port to use Intel `microcode-20130906.tgz` and
`amd-ucode-latest.tar.bz2` with internal file dates of 20130925.

So, I was happy to see that this port got updated and includes a tool that handles both Intel and AMD microcode files.

However, I was disappointed that the microcode files don't appear to be any newer than before.  It was using Intel
`microcode-20130222.tgz` and AMD `amd-ucode-2012-09-10.tgz`.  I actually don't currently have any FreeBSD systems using
AMD processors, but wanted to incorporate the latest amd-ucodes for completeness.

On Debian, the _latest_ files are the same in the source archive for `amd64-microcode-2.20131007.1+really20130710.1` and
the files have a date stamp of 20130907.  While all the files at [www.amd64.org/microcode](http://www.amd64.org/microcode)
have a date stamp of 20130927.

In checking the Intel site, found that the latest microcode file is 20140430.

Wonder what the newer revision for my CPU brings?

PR: ports/190712
