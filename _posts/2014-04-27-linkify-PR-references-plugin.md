---
title: "GitHub Pages Plugin to linkify PR: references?"
layout: post
comments: yes
category: other
---

After I had written many posts with _"PR:"_ lines everywhere with the internal
identification string assigned to my problem report submission.  It occurred
to me that it might be nice if it turned into the link to PR itself.

I looked to see if there might be a way to extend one of the markdown formats
to do this automagically, except I immediately realized that I'm kind of limited
to what 'github-pages' allows.

So, I decided the solution was to create some javascript code to do this.

This is what I came up with:

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
  var myRegExp = /PR: ([^\/]+\/\d+)/;
  var myMatch = myRegExp.exec(pHText);

  if (myMatch) {
    var myReplace = "PR: <a href=\"http://www.freebsd.org/cgi/query-pr.cgi?pr=" + myMatch[1] + "\" target=\"_blank\">" + myMatch[1] + "</a>";

    var pReplaced = pHText.replace(myRegExp, myReplace);
    pElement.innerHTML = pReplaced;
  }
});
</script>
{% endhighlight %}

Enjoy! :monkey_face:
