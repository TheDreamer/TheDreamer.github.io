---
title: devel/gdcm update fails when graphics/openjpeg is installed
layout: post
comments: yes
category: patches
---

`graphics/openjpeg` was recently updated to v1.5.2 and shortly after that upgraded to v2.1, with the v1.5.2 moved to
`graphics/openjpeg15`.  Since there were a few ports that needed it.  `devel/gdcm` is one of those ports.

Problem is that while both `graphics/openjpeg` and `graphics/openjpeg15` can co-exist on FreeBSD, since most ports are
working with the new openjpeg (2.1), there are some that aren't working.

On one of my systems:

     $ pkg info -r openjpeg
     openjpeg-2.1.0:
             nagstamon-0.9.9_2
             hfsexplorer-0.21_1
             rhino-1.7.r4
             ffmpeg0-0.7.16_2,1
             mencoder-1.1.r20140418
     $ pkg info -r openjpeg15
     openjpeg15-1.5.2:
             ffmpeg-2.1.1_4,1
             poppler-0.24.5_4
             gdcm-2.4.2_1
             mplayer-1.1.r20140418_1

While that looks weird, and portmaster is back to chugging through the backlog of ports to upgrade after the diversion to
get it do `devel/gdcm`.  The other 3 ports depending on 'openjpeg15' were updated after I had worked through the entries
in `/usr/ports/UPDATING`.

Now `devel/gdcm` seems to claim that it supports OPENJPEG_MAJOR_VERSION of 1 and 2.  It comes with source for 1.4.0 and
2.0.0.  Though the default is to use the v1 version, when a special advanced define that can be used to select the v2
version.  Alternatively, which is what is done through FreeBSD ports, is use the define to use the 'SYSTEM' openjpeg (which
also comes from ports.) :stuck_out_tongue_winking_eye:

In the first attempt to update this port, it failed because OPJ_UINT32 wasn't defined.  I eventually tracked it down in
`/usr/local/include/openjpeg-2.1/openjpeg.h`, it is not present in the 1.5.2 -- `/usr/local/include/openjpeg.h` header
file.  So, I thought its wanting the new openjpeg...so patch the Makefile to add `-I${LOCALBASE}/include/openjpeg-2.1` to
the `CFLAGS+= ` before the `-I${LOCALBASE}/include`, and update LIB_DEPENDS.

Well, it got further until it complained that there was no declaration for `opj_get_reversible()`, which is present in the
`openjpeg.h` for the bundled 2.0.0 version (_though it is strangely the only declaration without a comment block before it._)
Which might explain why trying to search backwards through the some 2800 svn revisions to the openjpeg source to find when
it was removed didn't turn up when it existed...and I gave up around revision 954.

Though I had pretty much already decided that the code didn't support v2.1, there was even a subversion revision message
about bumping the library version because its breaks ABI with previous among the revisions leading to the 2.1.0 release.
Though doesn't seem like this was among the reasons. 

So, it was how do I force it to use the `graphics/openjpeg15` port?  Given a comment that it wasn't possible to do
`find_package(OpenJPEG 2.0 REQUIRED)` in CMake.  So, things handle what ever is found.  And, in this case, CMake is finding
2.1.0, when we want 1.5.2.

Since LIB_DEPENDS 'promises' that `devel/openjpeg15` is present, forcing the outcome to use this version should be safe.
So, I opted to force it before CMake is invoked, as a modification to `files/patch-CMakeLists.txt` and the addition of
`files/patch-Source-Common-gdcmConfigure.h.in`.  Plus add a `post-patch:` to the Makefile to replace the `%%LOCALBASE%%`
put in by one of my patches to get the value from `${LOCALBASE}`.

A PR is pending, while things are read-only this weekend.
