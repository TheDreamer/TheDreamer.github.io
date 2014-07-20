---
title: "Toggling the base colors of my theme"
description: "Following the work I did on 'GitHub Pages Plugin to linkify references to ports?', the urge to fiddle with more javascript and css struck.  The color scheme of this site is based on Solarized: Precision colors for machine and people, where I had originally created this site based on having a light background."
layout: post
comments: yes
category: other
---

Following the work I did on
[GitHub Pages Plugin to linkify references to ports?]({% post_url 2014-07-20-linkify-PORT-references-too %}), the urge to
fiddle with more javascript and css struck.

The color scheme of this site is based on
[Solarized: *Precision colors for machine and people*](http://ethanschoonover.com/solarized), where I had originally created
this site based on having a light background."

However, with my vision problems ([cataracts][]), I don't do well with bright things anymore.  The explanation I've found is
that I have the [nuclear sclerosis][] form, which forms in the center of the lens, so pupil dialation gives me the
opportunity to see around the cataract. :sunglasses:

This seems to explain why when I'm self-conscious that I've noticed that I tend to appear to be looking to one side all the
time when I'm walking.

As a result of this, I have been adjusting the settings on some of my computers to be less bright, and had found a chrome
extension called [hacker vision][].  Though annoyingly for some reason the chrome webstore is immune to this extension, it
also doesn't do anything for `chrome://` or `file://` links.  My normal `homepage` is an html file of links that I've been
maintaining from back when my browser of choice was primarily netscape, before it had become IE for a while, and then
Firefox, etc.

Though I don't update it as much anymore, as I've been using [Xmarks][], though it since my departure from Firefox it hasn't
quite worked the same.  Namely I had Firefox extensions to use different folders as my bookmark bar and other profile tricks,
so the huge set of bookmarks sync'd by Xmarks had at least work and home profiles.  I had tried Xmarks profile option, but
it would frequently mess things up and I've had a few close calls of losing everything.  But, its working for now that I
haven't looked for an alternative.

Anyways, I tweaked the color settings of my `homepages`....

Which brings me to this site. (but not my other sites, yet.)  The Hacker Vision extension has its own scheme for 'inverting'
a sites color scheme.  But, its not the same as what can be done when using these *precision* colors.

I could swap out the CSS file for this page for ones that use a dark base scheme rather than the light base scheme (which
leaves the other colors of the site unaffected -- i.e. red stays red.)  Now there were two css files associated with this
site, and one was from a pair of `Solarized` files *For use with Jekyll and Pygments*...forget where I had found them, but
think it there's a github repo with for them.  The other file handles the rest of my site.  So I created a version of for
'dark'.  I used a sed line like this:

{% highlight bash %}
% sed -e 's/#002b36/_fdf6e3/g' -e 's/#073642/_eee8d5/g' -e 's/#586e75/_93a1a1/g' -e 's/#657b83/_839496/g' \
          -e 's/#839496/_657b83/g' -e 's/#93a1a1/_586e75/g' -e 's/#eee8d5/_073642/g' -e 's/#fdf6e3/_002b36/g' site.css | \
          sed -e 's/_/#/g' > site-dark.css
{% endhighlight %}

But, then it struck me that I should have a way to offer either scheme and have a control on the site to allow toggling
between the two.

Things got crazy after that....  most of the work was based on "[thesitewizard.com][]'s *How to Use Javascript to Change a
Cascacing Style Sheet (CSS) Dynamically*", with modifications to give things a personal touch in look and feel....  Which
was a combination of just trying things, Internet searches and using "Developer Tools" to debug the javascript.

The mess can be tracked down on github, so I won't bother will making this post any longer.... :stuck_out_tongue:

[cataracts]:            http://en.wikipedia.org/wiki/Cataract	
[nuclear sclerosis]:    http://en.wikipedia.org/wiki/Nuclear_sclerosis
[hacker vision]:        https://chrome.google.com/webstore/detail/hacker-vision/fommidcneendjonelhhhkmoekeicedej
[Xmarks]:               http://www.xmarks.com
[thesitewizard.com]:    http://www.thesitewizard.com/javascripts/change-style-sheets.shtml
