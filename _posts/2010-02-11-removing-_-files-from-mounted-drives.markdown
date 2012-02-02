---
layout: post
title: Removing ._ Files From Mounted Drives
permalink: /2010/02/removing-_-files-from-mounted-drives
tags:
- geek
- mac
- unix
---

Here's a quick snippet of goodness to remove those annoying '._' files that Macs generate on
mounted drives...

{% highlight bash %}
cd [wherever you want to clean]
find . -name "._*" -exec rm '{}' \; -print
{% endhighlight %}
