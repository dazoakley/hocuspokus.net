---
layout: post
title: When Using Apache As A Reverse Proxy To Passenger...
permalink: /2012/01/when-using-apache-as-a-reverse-proxy-to-passenger
tags:
- apache
- passenger
---

**ALWAYS** remember to set:

{% highlight apache %}
ProxyPreserveHost on
{% endhighlight %}

**Especially** if you're using the hostname to allow nginx to decide which
passenger app to run - that was a few hours of my life wasted yesterday
because of this...

