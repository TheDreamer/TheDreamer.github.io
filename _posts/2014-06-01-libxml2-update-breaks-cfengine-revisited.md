---
title: "Revisiting: libxml2 update breaks my cfengine"
layout: post
comments: yes
category: patches
---

On, April 20th, I wrote how the libxml2 update broke my cfengine, due to cf-agent not being able to find the libxml2.so
library after its version changed.  And, that I addressed the problem by partially linking cf-agent (and cf-promises)
statically.  This involved using the libtool option '-static-libtool-libs'.  To cause cf-agent and cf-promises to get
linked against the static libraries for the libraries have libtool definitions.  In this case, libpromises, libxml2 and
libiconv.

Things had been working fine at home, all builds having been done on April 19th.

But, recently I added a 4th FreeBSD system under the control of my CFEngine 3.x setup, initially I built it all shared
because its a CFEngine policy to apply various patches to ports, so that they'll build right.  It was the increasing
complexity of applying patches to ports so that they build correctly, while PRs I've submitted get no attention.  That
prompted the additional of this 4th host, so that I can just have CFEngine take care it automatically.  What's special
about the 4th host is that it is mobile.  I thought my first mobile CFEngine host would be Ubuntu, but I haven't used my
Ubuntu laptop in a long time...though again I really should get to it so that when I very likely replace it, it'll be
easier to get things back to how I like things :innocent:

Now, I'm probably far from getting that system fully under CFEngine's control, given that its been running for a few years
now.  Having started out life as a FreeBSD 9.0 system, then becoming a 9.1 and is currently a 9.2 system.  In fact, it
could've been older than my policy server....except that it had gotten redone a few times during its life as 9.0, including
a few times where I had unrecoverable zpool damage.

So, as a result of trying to get it into CFEngine's control, I held off updating ports, as there were alot where I would
need to reapply various patches first...which just reminded me to try to find more free time to get it going.  Plus I
was also making some big structural changes to my CFEngine setup, in part due to this.  And, breaking update.cf at one
time, which was unrecoverable since I had used the old convention of failsafe.cf calls update.cf.  Instead of the newer
where before making changes to (a working) update.cf, it is copied to failsafe.cf.  Meaning, failsafe.cf is now completely
standalone, while update.cf is called through regular execution.  Which does on occasion lead to problems where I use
stdlib routines in update.cf, which can't be found when its running as failsafe.cf.

Meanwhile, I couldn't get cfengine35 to build STATIC or PARTIAL.  Concluded that something had silently been updated in
ports that wasn't impacting my local systems yet.

Today, I got to digging...and discovered that it suddenly didn't know that there were dependencies to libxml2, specifically that it needed to also take a look at `/usr/local/lib/libiconv.la`.

Which I eventually tracked down as a big change that went in on April 23rd.

>  When linking a library libA with a library libB using libtool, if libB.la
>  exists, libtool will add all libraries libB.la refers to (dependency_libs
>  field) to the linker command line and store them in the dependency_libs
>  field of libA.la.  So everything that subsequently links with libA will also
>  link to these extra libraries.  This causes too much overlinking.
>  
>  This commit modifies Mk/Uses/libtool.mk so it empties the dependency_libs
>  field in .la libraries during staging.  However, because .la libraries have
>  very limited use when dependency_libs is empty it makes sense to completely
>  remove them during staging.
>  
>  So with this commit USES=libtool is modified to remove .la libraries and a
>  new form (USES=libtool:keepla) is introduced in case they need to be kept
>  (dependency_libs is still emptied).
>  
>  PORTREVISION is bumped on all ports with USES=libtool that install .la
>  libraries.  Most ports are also changed to add :keepla because .la
>  libraries have to be kept around as long as there are dependent ports with
>  .la libraries that refer to them in their dependency_libs field.  In most
>  cases :keepla can be removed again as soon as all dependent ports that
>  install .la libraries have some form of USES=libtool added to their
>  Makefile.
>  
>  PR:		ports/188759

Well, that's why libtool is failing to do handle its static build options, it doesn't know the dependencies of the
static library anymore which may also need to be linked in static.  Unlike where at runtime, the loader can trackdown the
dynamic library dependencies of the dynamic libraries that the executable depends on, etc.

Because of this modification, this happened to `/usr/local/lib/libxml2.la`:

{% highlight diff linenos %}
*** libxml2.la.orig	2014-04-17 19:12:53.000000000 -0500
--- libxml2.la	2014-05-24 19:20:49.014187000 -0500
***************
*** 1,5 ****
  # libxml2.la - a libtool library file
! # Generated by libtool (GNU libtool) 2.4
  #
  # Please DO NOT delete this file!
  # It is necessary for linking the library.
--- 1,5 ----
  # libxml2.la - a libtool library file
! # Generated by libtool (GNU libtool) 2.4.2
  #
  # Please DO NOT delete this file!
  # It is necessary for linking the library.
***************
*** 8,14 ****
  dlname='libxml2.so.2'
  
  # Names of this library.
! library_names='libxml2.so.2.8.0 libxml2.so.2 libxml2.so'
  
  # The name of the static archive.
  old_library='libxml2.a'
--- 8,14 ----
  dlname='libxml2.so.2'
  
  # Names of this library.
! library_names='libxml2.so.2.9.1 libxml2.so.2 libxml2.so'
  
  # The name of the static archive.
  old_library='libxml2.a'
***************
*** 17,31 ****
  inherited_linker_flags=''
  
  # Libraries that this one depends upon.
! dependency_libs=' -lz -L/usr/lib -llzma -L/usr/local/lib /usr/local/lib/libiconv.la -lm'
  
  # Names of additional weak libraries provided by this library
  weak_library_names=''
  
  # Version information for libxml2.
! current=10
! age=8
! revision=0
  
  # Is this an already installed library?
  installed=yes
--- 17,31 ----
  inherited_linker_flags=''
  
  # Libraries that this one depends upon.
! dependency_libs=''
  
  # Names of additional weak libraries provided by this library
  weak_library_names=''
  
  # Version information for libxml2.
! current=11
! age=9
! revision=1
  
  # Is this an already installed library?
  installed=yes
{% endhighlight %}

The main difference being __`dependency_libs`__, its resulting in the link failing because it can't resolve various
libiconv functions.

Guess I'll discard the STATIC link option for the port, but try to get PARTIAL working again.

Wonder if should update my cf-agent/cf-promises builds to use the newer libxml2?

Wonder how much of the old dependency_libs line do I need to patch into the Makefile?  Guess I'll just add it all...
though its mainly the `libiconv.la` file that is missed.

Also noticed that cf-promises is using libpromises.so, rather linking statically with it.  Seems I didn't notice that
the Makefile references AM_LDFLAGS, it doesn't define it anywhere.  So trying to have sed insert the '-static-libtool-libs'
option in to this variable, doesn't succeed.  Guess I'll insert it into 'LDADD='

Now I guess I will be re-installing CFEngine 3.5.3....

Also wonder if I should open a PR?

But, first... :zzz: before I leave for KUMC :hospital: tomorrow ... :worried:
