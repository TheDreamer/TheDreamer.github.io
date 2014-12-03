---
title: patch audio/espeak for Gnome3's pulseaudio
layout: post
comments: yes
category: patches
---

As part of the sudden loss of Gnome2 and the appearance of Gnome3, `audio/pulseaudio` got a significant bump in version
(0.9.23 -> 5.0)  This resulted in (re)build failure for `audio/espeak` if something other than the default option of
PORTAUDIO is selected.  RUNTIME includes both `audio/portaudio` and `audio/pulseaudio` as LIB_DEPENDS.

PULSEAUDIO is now making use of features that require C99 to succeed.  So, at first I thought it was a simple matter of
telling it to pass a switch to use c99 or c++99 or gnu99 or gnu++99(?), which would either involve using `USE_CXXSTD=gnu99`
or `USE_CXXSTD=gnu++99`, the first applies to C not C++ and the second doesn't exist, though was passed anyways so that
things would fail differently.  Had opted to use the new OptionsNG way, so it was:

	PULSEAUDIO_USE=	CXXSTD=gnu99

Or some variation of the sort.  Along the way, I considered that perhaps the issue was the base GNU compilers weren't new
enough (later found a document indicating that one of the things it had complained about first appeared in 4.3, and the
base compilers are 4.2.1.)  So the attempts became:

	PULSEAUDIO_USE= GCC=yes CXXSTD=gnu++99

Still no progress, And, new and different errors to confuse and baffle me.  Had introduced typos in places.  Eventually,
I looked through the files under `/usr/ports/Mk` to figure out what I should use.  I settled on:

	PULSEAUDIO_USES= compiler:c++0x

Success!

Next were some final things that `poudriere testport` complained about, resulting in making some changes to the `pkg-plist`
file.  So, now to send it off into the world....

PR: 195264
