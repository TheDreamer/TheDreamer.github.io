---
title: fix build of emulators/pipelight in poudriere
date: 2015-01-14 06:37:00
layout: post
comments: yes
category: patches
---

Due to the depth of support for NPAPI plugins by `www/chromium`, I've been looking for alternatives to restoring some of the lost
plugin functionality.  During the search, I had come across a pointer to `multimedia/moonlight`.  Namely by trying to join a
Microsoft Lync meeting, which requires Silverlight (which I had partly been able to do with an earlier chromium release), and in
trying to have it give me an installer, it suggests I look for Moonlight.  Which may or may not still be a supported offering on
Linux, but had dropped out of ports back in 2012.

Meanwhile, in earlier searches, I had come across references to `emulators/pipelight`.  And, thought I had already installed it,
or was at least already building it in *poudriere*.  I discovered that while I had created a UFS zvol for `~/.wine-pipelight`,
and induced a number of system panics due to trying to extend my snapshot backup scripts to handle this.  But, since its not in
use yet, currently skipping for now.  Though someday I'll need to, as I also have UFS zvol filesystems that I will eventually want
to back up.  The other being bounced around using CARP/HAST which I haven't had time to continue working on that project (since
getting CARP working....)

The *pipelight* project eing described as __"Windows plugins in Linux browsers"__, where FreeBSD is listed as a supported system. :grinning: Originally was mainly looking for Flash, but Silverlight is on the list.  Not sure about the others.

Thought what had diverted my attention for a while, was whether it was possible to install the Chrome for CentOS6 into the
linuxulator and have `www/chromium` access that.  but, there didn't seem to be a way to port acceptable way to wrap it, the
download is always the latest version, or a simple way to call it from chromium (have to parse the plugin's manifest to
generate some commandline flags to add to the browser's command line.  Given more time I'd probably have devised something, but
haven't got the first step of having it work.)  In an earlier time, I had wondered if it would be possible to use the Mac OS X
Flash on FreeBSD....and since learning that clang and LLVM are associated with Apple, it does raise the question of whether code
generated to use LLVM libraries/resources could be made to run with minimal effot by switching out the LLVM libraries?  Perhaps
my recollection is blurred by my past where (*commercial*) software products that were sufficient abstracted that support other
systems started by created new system specific libraries.  Where another product had been developed onto a proprietary 'xVM' (which I believe also had originated from the University of Illinois) was being pushed to conver it to use an industry recognized
language and use that xVM....namely Java & JVM.

Anyways, I found this port would not build in *poudriere*.  Where, eventually I tried to build it outside of *poudriere* where it
worked.  Causing me to finally start an itneractive bulk jail to see what was differnt inside of *poudriere* versus on the outside.
FOund the port uses gpg, but its set ot depend on gpg2 (which will in the absence of gpg, creats a symlink.  Evidently, not sufficiently compabitile for pipelight though.

So, I switched it to use `security/gnupg1`.

-----

A different change makes it in, which fixes the arguments to gpg as gpg2.  But, the question is my patch better or not to the
current committed change.  The question being how does the port use gpg as a RUN_DEPENDS, and will how it uses gpg work.

PR: 196708
