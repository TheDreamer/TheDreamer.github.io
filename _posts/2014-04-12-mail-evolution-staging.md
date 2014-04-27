---
title: mail/evolution got updated to do staging, but now its missing files
layout: post
comments: yes
category: patches
---

This port got updated to do staging, but now its missing files (its shareed
libraries).  pkg-plist didn't include them.  So, updated pkg-plist to include
all the files that got copied to staging.

PR: ports/188525

The maintainer had already fixed by the time he got around to my PR, though he
left out the gconf files.  Don't recall if I've ever looked at them before
so probably not an issue that their missing now that mail/evolution is being
staged.

Still not submitted is my [patch]({% post_url 2013-02-06-mail-evolution-hack %})
to address the annoying modal dialog for appointment reminders set to send emails.
