---
title: Unable to launch calibre from applications menu
layout: post
comments: yes
category: patches
---

It has apparently been a while since I've used this app, like evidently pre-gnome3 or was it just pre-9.3?  Having trouble
remembering wehn was the last time I used it.  Feels like it was just before my last KUMC trip in December.  But, don't recall
when I had the jump to gnome3...

But, clicking on it from Favorites wouldn't start it (from any of the locations that I can access this sub-menu, though I think
finding the gnome-shell-extension to put it on top bar was rather recent.)  It would start in terminal window, and its possible
that I may have done that pre-KUMC trip while my desktop was less usable than I've gotten it so far (it does have some useful
features which I had hoped running two screens & Xinerama would give me...but hasn't and causes other problems, which I keep
meaning to fix someday.  Will have to switch to TwinView, since gnome3 seems to require Composite which is disabled when using
Xinerama.

Eventually, it occurred to me that it might be doing something different from menu than starting it naked on commandline.

Yup.  The `calibre-gui.desktop` file has `Exec=calibre --detach %F`.  I tried `calibre --detach` in a terminal window.  Result:

        Usage: calibre [opts] [path_to_ebook]

        Launch the main calibre Graphical User Interface and optionally add the ebook at
        path_to_ebook to the database.


        Whenever you pass arguments to calibre that have spaces in them, enclose the arguments in quotation marks.

        calibre: error: no such option: --detach

And, it exits.  Hmmm, do an online search.  Online man page has net to the option __(linux only)__, guess we're not linux anymore...

Remove the `--detach` from the `calibre-gui.desktop` file -- update the desktop database and restart gnome-shell -- and try
starting it from the menu.  It starts.  Now to figure out where its coming from, since there's no files contain '.desktop' in the
source.

Seems to involve patching `linux.py`....

PR: 196817
