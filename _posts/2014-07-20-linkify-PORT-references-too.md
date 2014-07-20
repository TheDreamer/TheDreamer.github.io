---
title: "GitHub Pages Plugin to linkify references to ports?"
layout: post
comments: yes
category: other
---

Following my previous post,
[GitHub Pages Plugin to linkify PR: references?]({% post_url 2014-04-27-linkify-PR-references-plugin %}),
on June 8th, I updated the URL generated to point to the new [bugzilla](https://bugs.FreeBSD.org/bugzilla) site.  I then
made a mod on June 14th to handle `PR:` lines with just numbers following it.  Though I didn't get that mod working until
June 26th when I created a post using the syntax.

The latest ugly hack ~~is~~was:

{% highlight javascript linenos %}
<script type="text/javascript">
function addEvent(evType, f) {
  if (window.addEventListener){
    window.addEventListener(evType, f, false);
    return true;
  } else if (window.attachEvent){
    var r = window.attachEvent("on"+evType, f);
    return r;
  } else {
    return false;
  }
}
addEvent('load', function() {
  var pElements = document.getElementsByClassName('body');

  var pElement = pElements.item(0);
  var pHText = pElement.innerHTML;
  var myRegExpO = /PR: ([^\/]+)\/(\d+)/;
  var myMatchO = myRegExpO.exec(pHText);

  var myRegExpN = /PR: (\d+)/;
  var myMatchN = myRegExpN.exec(pHText);

  if (myMatchO) {
    var myReplaceO = "PR: <a href=\"https://bugs.FreeBSD.org/bugzilla/show_bug.cgi?id=" + myMatchO[2] + "\" target=\"_blank\">" + myMatchO[1] + "/" + myMatchO[2] + "</a>";

    var pReplacedO = pHText.replace(myRegExpO, myReplaceO);
    pElement.innerHTML = pReplacedO;
  }
  if (myMatchN) {
    var myReplaceN = "PR: <a href=\"https://bugs.FreeBSD.org/bugzilla/show_bug.cgi?id=" + myMatchN[1] + "\" target=\"_blank\">" + myMatchN[1] + "</a>";

    var pReplacedN = pHText.replace(myRegExpN, myReplaceN);
    pElement.innerHTML = pReplacedN;
  }
});
</script>
{% endhighlight %}

But, since I had evolved my 'new packages' alias from:

    pkg_version -vIl\<

to

    pkg version -vIl\<

to a script:

{% highlight tcsh %}
set D=`mktemp -d`
mkfifo ${D}/pkg-version-out
pkg version -vIl\< | tee ${D}/pkg-version-out | cut -f 1 -d ' ' | xargs -n 1 pkg info -o | awk '{ printf "                    http://freshports.org/%s\n", $2 }' | paste -d '\n' ${D}/pkg-version-out -
rm -fr ${D}
{% endhighlight %}

Which produces output like this:

{% highlight text %}
bitlbee-3.2.1_2                    <   needs updating (index has 3.2.2)
                    http://freshports.org/irc/bitlbee
curl-7.37.0                        <   needs updating (index has 7.37.1)
                    http://freshports.org/ftp/curl
gcc-4.7.3_1                        <   needs updating (index has 4.7.4)
                    http://freshports.org/lang/gcc
i386-wine-devel-1.7.21,1           <   needs updating (index has 1.7.22,1)
                    http://freshports.org/emulators/i386-wine-devel
openjdk-7.60.19_2,1                <   needs updating (index has 7.65.17,1)
                    http://freshports.org/java/openjdk7
p5-Email-Address-1.90.0            <   needs updating (index has 1.90.5)
                    http://freshports.org/mail/p5-Email-Address
p5-IO-Socket-SSL-1.997             <   needs updating (index has 1.997_1)
                    http://freshports.org/security/p5-IO-Socket-SSL
py27-setuptools27-5.1              <   needs updating (index has 5.4.1)
                    http://freshports.org/devel/py-setuptools27
{% endhighlight %}

Where the URLs are clickable in `gnome-terimal` :grin:  And, http://freshports.org is where I want to link to, so I can
see what's new in a given port.  Since on occasion there have been some questionable port updates.

~~~~~~~~~~ text
Like a recent chase of portrevision bump for sqlite3.  Don't know why they couldn't have done a
late UPDATING entry, instead of bumping ports slowly over the weeks following the port's update.
I had already run into a port unhappy about the update, so I had already rebuilt everything that
depends on sqlite3.

Or the portrevision bumps to make neuter libtool, so that its useless for doing development on
FreeBSD.  Unless the plan is move to Linux of separate `-dev` packages?  Or portrevision bumps
from stagifing the port (especially when it breaks or exposes that the original pkg-plist is bad.)
~~~~~~~~~~

Anyhoo.... It came to me last night.... could I turn the markdown patterns of <code>\`<category\>/<port-name\>\`</code>
into links?

I renamed my include file from `pr-handler.html` to `auto-handler.html` and added this block of code to it:

{% highlight javascript linenos linenostart=36 %}
addEvent('load', function() {
  var pElements = document.getElementsByClassName('body');

  var pElement = pElements.item(0);
  var pHText = pElement.innerHTML;
  var myRegExpO = /<code>((?!files)[a-z0-9\-]+)\/([^A-Z<]+)<\/code>/;
  var myMatchO;
  while ((myMatchO = myRegExpO.exec(pHText)) !== null) {
    var myReplaceO = "<code><a href=\"http://freshports.org/" + myMatchO[1] + "/" + myMatchO[2] + "\" target=\"_blank\">" + myMatchO[1] + "/" + myMatchO[2] + "</a></code>";

    pHText = pHText.replace(myMatchO[0], myReplaceO);
  }
  pElement.innerHTML = pHText;
});
{% endhighlight %}

*The regex is ugly and probably incomplete, but seems to work for the sample set of existing posts....*

Enjoy! :monkey_face:
