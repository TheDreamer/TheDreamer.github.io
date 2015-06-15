---
title: A quick update to 'print/cloudprint'
layout: post
comments: yes
category: patches
---

While browsing through ports looking for something else, I stopped to take a peek at `print/cloudprint`.  Now I've been
building this port for sometime, with the intent of trying to get it set up.  As I am currently doing this through the small
ubuntu VM running on my main system 'zen', I named the VM 'uzen'.

Originally, the 'u' was for 'ubuntu', though I often think of it as 'u' for 'micro'.  Not totally sure what I was thinking
when I came up with it originally.  While keeping with the theme, naming the VM '[slave][1]' would seem appropiate in a
number of ways, yet inappropriate in others.  And, the other computer from the show '[orac][2]', is already taken/reserved
for what was my old (main) ubuntu system (back when '[Zen][3]' - Windows 7 was dead, and then resurrected as '[zen][3]'
running FreeBSD 9.0 [now 9.3]) I'm thinking of resurrecting it someday, possibly to run FreeBSD 10+, even though its only a
Core2 Quad machine....)  Still no idea on what to do with the pile of parts of the what had evolved into my intended FreeBSD
10+ server. (its a U1 server that I'm turning into a tower...)

So, while getting deep into this port, hopefuly I'll work my way back to the original task... :confused:  I found that it
had recently been updated to 0.12, and the main addition is OAUTH2 support (and the removal of REST) due to Google turning
off the old method.

A more or less quick update of the port was done...

PR: 200878

[1]: https://en.wikipedia.org/wiki/Scorpio_(Blake%27s_7)#Slave
[2]: https://en.wikipedia.org/wiki/Orac_(Blake%27s_7)
[3]: https://en.wikipedia.org/wiki/Zen_(Blake%27s_7)
