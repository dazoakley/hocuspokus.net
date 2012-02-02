---
layout: post
title: Installing Apache and PHP Troubles
permalink: /2007/11/installing-apache-and-php-troubles
tags:
- apache
- linux
- php
- ubuntu
---

I just recently set Apache up on my home server (more on the server at some point in the future), but I was
having problems serving up php pages. Every time I tried to access a php based page, Firefox came up asking
if I wanted to download a '.PHTML' file!!! :(

Thankfully the answer (like most things with Ubuntu) was found on the
[Ubuntu Forums](http://ubuntuforums.org/showthread.php?t=209120)...

Simply edit the file `/etc/apache2/apache2.conf` by adding the following line:

{% highlight apache %}
AddType application/x-httpd-php .php .phtml
{% endhighlight %}

Now php files should be handled by the server in the way that they were intended.
