---
title: "sysutils/nut is causing my system to panic during boot"
layout: post
comments: yes
category: other
---

Being crazy, I decided to run `freebsd-update install` from work on my home computer, to take it up to 9.2-FreeBSD-p10.
I rebooted and it never came back.

This is bad, since I depend on it for access to my email currently.  One of these days I'll move it back into the cloud.

When I got home, the system was stuck in a bad way, but I was able to convince it to find all its zpools from single user
mode and try again.  Except it soon started acting weird during the boot and eventually PANIC'd.  During the work in trying
to fix things, I discovered that it had done 3 of these reboot/PANIC's when I had first rebooted it...before it ended up
in a bad way with its zpools.

The backtrace seemed to point at VirtualBox's kernal module as the problem, I tried rebuilding it while in single user.
But, it didn't help.  I tried rolling-back the freebsd-update.  Which hung.  But, seemed to complete.  Didn't help.  I
then disabled everything virtualBox.  That worked, except that I needed something from my Dropbox and that's running in
a VM normally.

Also, my desktop was looking kind of weird as `metacity` was refusing to run.  While digging in to see why, found that
it didn't like the state of something in /dev.

Hmmm, just about everything has a gid of `_ups`.  That's not right.  Fixing things by hand as best I could (and using work
FreeBSD workstation as reference) got things working again.  But, rebooting....things were messed up again.

I had created the restore gid's as a script, so boot single user and run my script and then let it continue....that seemed
to help.  but, why was it going wrong.

Start tracking backwards on what I might have done or might have updated ports wise.

Among things I had done, was to drop in my own file into `/usr/local/etc/devd`.  But, that file wasn't the culprit.  Finally
I looked in the directory, and there was a 'new' file in there I hadn't seen before.  `nut-usb.conf`...hmmm.

Lots of blocks like this:

{% highlight text %}
notify 100 {
        match "system"          "USB";
        match "subsystem"       "DEVICE";
        match "type"            "ATTACH";
        match "vendor"          "0x0001";
        match "product"         "0x0000";
        action "chgrp _ups /dev/$device-name*; chmod g+rw /dev/$device-name*";
};
{% endhighlight %}

Skimmed the manpage:

=Variables that can be used with the match statement=
    A partial list of variables and their possible values that can be used together with the match statement.

Early in the manpage it had noted that:

=device-name "string";=
    This is shorthand for 'match "device-name" "string"'.  This matches a device named __string__, which is allowed
to be a regular expression or a variable previous created containing a regular expression.  The "device-name"
variable is availabe for later use with the action statement.

The examples where $device-name is used are:

{% highlight text %}
attach 0 {
        device-name "(ath|wi)[0-9]+";
        action "/etc/pccard_ether $device-name start";
};

detach 0 {
        device-name "(ath|wi)[0-9]+";
        action "/etc/pccard_ether $device-name stop";
};
{% endhighlight %}

Well, those 'notify' blocks don't have _match_ statements for "device-name"...  so that would suggest that its doing:

{% highlight text %}
chgrp _ups /dev/*; chmod g+rw /dev/*;
{% endhighlight %}

That looks a lot like what its doing, seems to happen early in the boot when the OS in looking at what devices are
already attached by USB.  On my home workstation, I have a USB attached UPS (at work its serial attached.)

*There's no UPS communication for my other two FreeBSD systems, as they are each connected to a PowerSource 400.  And,
hopefully, they'll stay up for hours...and get through most outages.  Though between the pair, there isn't quite enough
infrastructure to make it that useful....yet :wink:*

The devd file (generated from Linux for FreeBSD) was introduced to the nut version 2.7 line.  The port was upgraded from
v2.6.5 to 2.7.2.  The port maintainer's UPS is broken, so he wasn't able to test the USB feature of the update and can't
test the issue that I've reported.

PR: 191777

-----

Saturday, July 19 2014

The ~~current fix, is to remove the `'*'` from the nut-usb.conf file~~ latest fix, is to change `$device-name*` to `$cdev`.
My fix would probably be to have CFEngine promise the file's non-existence, though I wondered if I want to figure out the
correct fix to have CFEngine make (or submit a patch....) *Wonder if the `$cdev` fix is correct?*

~~Though I don't think there's any harm in it change the group of /dev to be '_ups'  Assuming devfs allows this.  Though
it'll look kind of weird with the permissions of /dev change from `'dr-xr-xr-x'` to `'dr-xrwxr-x'`, but again probably
won't cause problems.  At least not the kind where things don't work or it causes my system to panic.~~

The reason this file isn't necessary for me, is that inorder to get **nut-2.6.5** working in the first place, I had put
the `_ups` user into the `usb` group.

The intent of this devd would be to make specific usb device node to have a group of `_ups`, and be group read-write-able.

For another application, VirtualBox, I had made all the *usb* device nodes to be `'0660'` and have a group of `'usb'`.
Though the lack of USB 2.0 support has limited the usefulness of this, which I've solved by getting a
**'Silex SX-DS-4000US'**.

In trying to understand devd, I looked at the source and found this:

{% highlight c++ %}
        // Since we can be called from both a device attach/detach
        // context where device-name is defined and what we want,
        // as well as from a link status context, where subsystem is
        // the name of interest, first try device-name and fall back
        // to subsystem if none exists.
        value = c.get_variable("device-name");
        if (value.length() == 0)
                value = c.get_variable("subsystem");
{% endhighlight %}

And, further down for process_event....only the attach/detach events have a special set_variable "device-name" action.  Does
that prove its not part of of "notify"?

{% highlight c++ %}
process_event(char *buffer)
{
        char type;
        char *sp;

        sp = buffer + 1;
        if (Dflag)
                fprintf(stderr, "Processing event '%s'\n", buffer);
        type = *buffer++;
        cfg.push_var_table();
        // No match doesn't have a device, and the format is a little
        // different, so handle it separately.
        switch (type) {
        case notify:
                sp = cfg.set_vars(sp);
                break;
{% endhighlight %}

...

{% highlight c++ %}
        case attach:    /*FALLTHROUGH*/
        case detach:
                sp = strchr(sp, ' ');
                if (sp == NULL)
                        return; /* Can't happen? */
                *sp++ = '\0';
                cfg.set_variable("device-name", buffer);
                while (isspace(*sp))
                        sp++;
                if (strncmp(sp, "at ", 3) == 0)
                        sp += 3;
                sp = cfg.set_vars(sp);
                while (isspace(*sp))
                        sp++;
                if (strncmp(sp, "on ", 3) == 0)
                        cfg.set_variable("bus", sp + 3);
                break;
{% endhighlight %}

*Probably not, would need actually know what is and isn't in 'buffer'.*

This is making my head hurt....and totally distracting me from what I had intended to try to get done today.

*Should probably decide whether the fix is right before I run into a reboot, or whether to go with the CFEngine promise....*
