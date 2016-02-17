---
layout: post
title: Sync Your Things Database via Dropbox
tags:
- dropbox
- gtd
- mac
- sync
- things
- to-do
- tutorial
---

[Dropbox](https://www.getdropbox.com/) is a great service, I'm using it happily to keep my files in sync
across multiple computers - I'm even using it to keep all of my [passwords in sync](http://www.switchersblog.com/2008/10/1password-29-br.html), but I've thought of another great use...
How about syncing my [Things](http://culturedcode.com/things/) database between my macs (as this is my to-do
list manager of choice)?

This is not fully tested yet, (just thought of it this morning) so i'll update a bit later and report on as
to wether things goes completely mental, but the way I've done this is as follows...

Make sure Things is completely shut down, then open up a terminal and type in the following commands:

{% highlight bash %}
cd ~/Library/Application\ Support/Cultured\ Code/
{% endhighlight %}

This moves us into the correct directory. First, to be on the safe side - we'll take a backup of our files...

{% highlight bash %}
cp -R Things Things.bak
{% endhighlight %}

Now just move the Things directory into your dropbox and create a symbolic link in its place.

{% highlight bash %}
mv Things ~/Dropbox/
ln -s ~/Dropbox/Things Things
{% endhighlight %}

Fingers crossed this should have the desired results! :)

**Update:** It works!!! :D On the second computer all you need to do to get the ball rolling is to close down
Things, open up a terminal and type the following commands:

{% highlight bash %}
cd ~/Library/Application\ Support/Cultured\ Code/
rm -rf Things
ln -s ~/Dropbox/Things Things
{% endhighlight %}

Note - my 2nd mac only had a fresh install of Things - **no data**. I installed it, opened it up (so the
initial database was created), then did the above.
