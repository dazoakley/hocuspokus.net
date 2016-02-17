---
layout: post
title: When Using Apache As A Reverse Proxy To Passenger...
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
