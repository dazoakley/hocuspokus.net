---
layout: post
title: Give Your Website a Homepage Button on iOS
permalink: /2013/02/give-your-website-a-homepage-button-on-ios
tags:
- webapp
- ios
---

I had a few quiet days this week so I thought I'd have a play about with giving
[SciCombinator](http://www.scicombinator.com) it's own iOS icon and make it feel
a little more like a first class iOS app.  This turned out to be pretty simple to
be honest; here's the steps involved and the small amount of code needed...

First up, icons.  Generate your app icon in the following sizes (in pixels):

* 57x57 - non-retina iPhone and iPod Touch
* 72x72 - non-retina iPad
* 114x114 - retina iPhones and iPad mini
* 144x144 - for the iPads 3 &amp; 4

No need to make them look like actual iOS icons - iOS can apply the rounded corner
and reflection effects for you (unless you want a **really** specific look for
your icons, then go ahead).  I just made nice, flat, square images...

Save these icons into your app/site so that they'll get served up, and then add the
following tags into your site header:

{% highlight html %}
<link rel="apple-touch-icon" href="/apple-touch-icon-57x57.png" />
<link rel="apple-touch-icon" sizes="72x72" href="/apple-touch-icon-72x72.png" />
<link rel="apple-touch-icon" sizes="114x114" href="/apple-touch-icon-114x114.png" />
<link rel="apple-touch-icon" sizes="144x144" href="/apple-touch-icon-144x144.png" />
{% endhighlight %}

The important thing to note here is the "sizes" attribute - this lets iOS (mobile safari)
know which icon to use for the best look.  Also, if you don't want iOS to apply the
rounded corners and reflection to your icon, you can set the "rel" attribute to
"`apple-touch-icon-precomposed`" - this will tell iOS leave them as they are.

Just below these link tags, add the following meta tags:

{% highlight html %}
<meta name="apple-mobile-web-app-capable" content="yes" />
<meta name="apple-mobile-web-app-status-bar-style" content="black" />
<meta name="format-detection" content="telephone=no" />
{% endhighlight %}

These tell mobile safari the following:

1. This website can be run in "standalone" mode - with no mobile safari chrome
2. Set the colour of the iOS status bar (the top) to black
3. Do not attempt detect telephone numbers on the page (unless this is appropriate for your site)

See [here](http://developer.apple.com/library/safari/#documentation/AppleApplications/Reference/SafariHTMLRef/Articles/MetaTags.html)
for more information on the various meta tags you can pass into mobile safari.

This is all you need to do to get your website/app to behave almost like a standalone app
on iOS.  Now for the final wrinkle...  If you open up your site in standalone mode from
your glorious new homepage icon, every link you touch; be it internal, or external to your site
will open up in mobile safari rather than staying within your app.

Thankfully, some other people have thought of this one!  All you need to do is add in a little
but of javascript to intercept internal links within your site, stop the default link action,
and just change the URL via javascript.  Here's two decent examples I used to come up with the
solution for [SciCombinator](http://www.scicombinator.com):

* [https://gist.github.com/kylebarrow/1042026](https://gist.github.com/kylebarrow/1042026)
* [https://gist.github.com/irae/1042167](https://gist.github.com/irae/1042167)

And that's all there is to it!

