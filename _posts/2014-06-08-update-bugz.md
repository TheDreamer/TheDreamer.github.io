---
title: Update devel/pybugz to 0.10.1
layout: post
comments: yes
category: patches
---

In the announcement of FreeBSD switch from GNATS to Bugzilla, there was reference to this port.  And, in the source and
readme was reference to ~/.bugzrc.  Except the ports version (0.9.3) there's nothing in the code to use a config file.

So, I looked to see what the latest on github was, and then worked on updating the port to fetch the latest from github
directly and see if that would work better.

Well, it took some fiddling around to figure out that setting one or more status conditions is **mandatory** for search
to return any results.  And, figuring out what to put for the various options isn't simple.  Would be nice if there was
a simple way to search for ALL except `'Issue Resolved'`, in case my interest is only see if there's another bug in some
open state that matches what I want to report.

Also not very obvious that the different `status` codes are: 'Needs Triage', 'Open', 'Approval Needed', 'Commit Ready',
'In Discussion', 'Needs MFC', 'Issue Resolved', 'Timeout', 'UNCONFIRMED'.  Or that for `Product` the values are:
'Administrative Concerns', 'Base System', 'Documentation' (_instead of docs_), or 'Ports Tree' (_instead of ports_).  Plus
what were just 'Categories' has now become 'Product' and 'Component'.

I think this is how things map now:

| Categories | Product | Component |
|:----------:|:------- |:--------- |
|advocacy|Documentation|Advocacy|
|alpha|Base System|alpha|
|amd64|Base System|amd64|
|arm|Base System|arm|
|bin|Base System|bin|
|conf|Base System|conf|
|docs|Documentation|Documentation|
|gnu|Base System|gnu|
|i386|Base System|i386|
|ia64|Base System|ia64|
|java|Base System|java|
|kern|Base System|kern|
|misc|Base System|misc|
|ports|Ports Tree|Individual Port(s)|
|powerpc|Base System|powerpc|
|sparc64|Base System|sparc64|
|standards|Base System|standards|
|threads|Base System|threads|
|usb|Base System|usb|
| - |Base System|wireless|
|www|Administrative Concerns|Bug Tracker|
|www|Administrative Concerns|Forums|
|www|Administrative Concerns|Junk|
|www|Administrative Concerns|Mailing Lists|
|www|Ports Tree|Infrastructure|

But, before I got to trying to figuring out how to use `bugz` to 'post', I found in doing a search for 'pybugz', that
somebody else had already submitted a patch to update to 0.10.1.  They had a slightly different set of GH_* parameters,
which actually got the GH_COMMIT that matched what was on the github site for the version tag.  But, I compared the files
in the archives and they were identical.  Don't know why it did that.

Probably should update how I reference PR's, or something and adjust my pr-handler code.  For now it does take the issue
number out of the line and convert it to a link to Bugzilla site.

PR: ports/190795
