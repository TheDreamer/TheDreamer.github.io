---
title: "sysutils/panicmail: assist FreeBSD developers in identifying and fixing panics"
description: "Recently, I had yet another panic.  Couldn't remember how I had examined the previous vmcore, while searching online on how again, I came across this port.  Almost immediately, I started fiddling with it.  Like changing the From: address for autosubmit to use a proper email address...."
layout: post
comments: yes
category: patches
---

Recently, I had yet another panic.  Couldn't remember how I had examined the
previous vmcore, while searching online on how again, I came across this port.

Almost immediately, I made some tweaks to it.  Like change the From: address
for autosubmit to use a proper email address rather than my internal root
account.

I forget if I ever got around to creating a genericstable config for my
FreeBSD sendmail servers, which have replaced my Ubuntu postfix servers.
Though if I had, the address I would use in place of root on my primary
desktop would differ from the email I use for FreeBSD related correspondence.

Initially, I made the tweaks by hand and then coded up a CFE3 promise to
recreate it on my other FreeBSD servers.

{% highlight cfengine3 linenos %}
bundle edit_line add_panicmail_sendfrom
{
insert_lines:

        ": $(const.dollar){panicmail_sendfrom:=\"root\"}"
                location => after("^: \\$\{panicmail_sendto:=.*");

replace_patterns:

        "From: root"
                replace_with =>
                        AllWith("From: $(const.dollar){panicmail_sendfrom}");

}
{% endhighlight %}

Then, I started to set things up for possible PR submission to send up my
*enhancements*.  But, as I was doing that, I thought up other enhancements
and it got a little out of hand.  Now, I'm not sure I want to send it up....

new `rc.conf(.local)` entries (with defaults):

    panicmail_autonotify="YES"
    panicmail_sendfrom="root"
    panicmail_usecrashinfo="NO"
    panicmail_usekernel=""

`panicmail_autonotify` -- sets whether you want to be cc'd when
`panicmail_autosubmit="YES"`.

`panicmail_sendfrom` -- change From: to be something other than root.  I use:

    panicmail_sendfrom="Lawrence Chen <beastie@tardisi.com>"

`panicmail_usecrashinfo` -- if set to "YES", **crashinfo(8)** instead of just
a kgdb backtrace.

`panicmail_usekernel` -- use a kernel other than `"sysctl -n kern.bootfile"`.
When I compile my own kernel, it results in 3 files:  `'kernel'`,
`'kernel.symbols'`, and `'kernel.debug'`.

> From NOTES:

> DEBUG happens to be magic. 
> The following is equivalent to 'config -g KERNELNAME' and creates
> 'kernel.debug' compiled with -g debugging as well as a normal 'kernel'.
> Use 'make install.debug' to install the debug kernel but that isn't
> normally necessary as the debug symbols are not loaded by the kernel and
> are not useful there anyway.

But, for the kgdb backtrace the `'kernel.debug'` would probably be more useful.

----

In [revisiting]({% post_url 2014-04-13-sysutils-panicmail-revisited %}) the
continued fiddling with panicmail, and reviewing the manpage for **kgdb(1)**
and reading *"Optionally, the name of the kernel symbol file and the name of
the core dump file can be supplied on the command-line as positional arguments.
If no kernel symbol file name has been given, the symbol file of the currently
running kernel will be used."*  Led me to suspect that `'kernel.symbols'`
alone is sufficient for the kgdb backtrace.
