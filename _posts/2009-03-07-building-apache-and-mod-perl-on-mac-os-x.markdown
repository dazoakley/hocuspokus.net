---
layout: post
title: Building Apache and Mod Perl on Mac OS X
tags:
- apache
- mac
- perl
---

I've finally had my work laptop upgraded to Leopard!!! :)

As such, I've just spent the last couple of days getting things all set-up nicely so that I can get on with my
work. Most of the work that I do is web development orientated, and mainly using [Perl](http://www.perl.org/)
and [Catalyst](http://www.catalystframework.org/), so an install of Apache and mod_perl is needed.

OS X does come with a complete install of Apache (even with mod_perl!) out of the box and ready to go (info on
using this set-up can be found [here](http://bixsolutions.net/forum/thread-11.html)), but I'm also working on
another project that may involve the use of [Jaxer](http://www.aptana.com/jaxer), and this requires a newer
build of Apache than the one shipped with Leopard. :(

Thankfully building these tools isn't too complicated, here's a quick dump of my notes on getting this done.
Note, I'm installing them into [/usr/local](http://hivelogic.com/articles/2005/11/using_usr_local) so that I
don't mess with any of the OS X internals that I shouldn't be touching - this is completely removable.

First, make a work area for building:

{% highlight bash %}
sudo mkdir -p /usr/local/src
sudo chgrp admin /usr/local/src
sudo chmod -R 775 /usr/local/src
cd /usr/local/src
{% endhighlight %}

<h3>Apache</h3>

{% highlight bash %}
curl -O http://apache.mirror.infiniteconflict.com/httpd/httpd-2.2.11.tar.gz
tar zxvf httpd-2.2.11.tar.gz
cd httpd-2.2.11
{% endhighlight %}

Now here's the big one - the Apache configuration. This compiles a heap of modules I probably don't need, but
it's nice to have them there in case I do ever need them...

{% highlight bash %}
CFLAGS="-O3" CXXFLAGS="-O3" \
./configure --prefix=/usr/local/apache2 \
--enable-autoindex \
--enable-cache \
--enable-cgi \
--enable-deflate \
--enable-dir \
--enable-disk_cache \
--enable-fastcgi \
--enable-file_cache \
--enable-headers \
--enable-include \
--enable-info \
--enable-log_config \
--enable-log_forensic \
--enable-logio \
--enable-mem_cache \
--enable-mime \
--enable-mime_magic \
--enable-negotiation \
--enable-perl \
--enable-proxy \
--enable-proxy-balancer \
--enable-proxy-http \
--enable-rewrite \
--enable-speling \
--enable-status \
--enable-suexec \
--enable-userdir \
--enable-usertrack \
--enable-version \
--enable-vhost_alias \
--enable-so \
--enable-mods-shared=all
{% endhighlight %}

Then the standard make and install:

{% highlight bash %}
make
make test
sudo make install
{% endhighlight %}

Now to add some configuration so that Apache starts on system boot, first we need to create a startup script:

{% highlight bash %}
cd /System/Library/StartupItems/
sudo mkdir Apache
cd Apache
sudo touch Apache
sudo chmod a+x Apache
mate Apache
{% endhighlight %}

Paste this content into the file:

{% highlight bash %}
#!/bin/sh

##
# Apache HTTP Server
##

. /etc/rc.common

StartService () {
    ConsoleMessage "Starting Apache web server"
    /usr/local/apache2/bin/apachectl start
}

StopService () {
    ConsoleMessage "Stopping Apache web server"
    /usr/local/apache2/bin/apachectl stop
}

RestartService () {
    ConsoleMessage "Restarting Apache web server"
    /usr/local/apache2/bin/apachectl restart
}

RunService "$1"
{% endhighlight %}

Then a configuration file:

{% highlight bash %}
sudo touch StartupParameters.plist
mate StartupParameters.plist
{% endhighlight %}

Paste this content into the file:

{% highlight text %}
{
  Description     = "Apache web server";
  Provides        = ("Web Server");
  Requires        = ("DirectoryServices");
  Uses            = ("Disks","Network Time");
  OrderPreference = "None";
}
{% endhighlight %}

Then reboot and open up [http://localhost](http://localhost) to make sure things have worked.

<h3>Mod Perl</h3>

This is a lot more straight forward:

{% highlight bash %}
cd /usr/local/src
curl -O http://perl.apache.org/dist/mod_perl-2.0-current.tar.gz
tar zxvf mod_perl-2.0-current.tar.gz
cd mod_perl-2.0.4
perl Makefile.PL MP_AP_PREFIX=/usr/local/apache2
make
make test
sudo make install
{% endhighlight %}

Now all you have to do is add the following line to your Apache httpd.conf (`/usr/local/apache2/conf/httpd.conf`)
with all of the LoadModule entries:

{% highlight ini %}
LoadModule perl_module modules/mod_perl.so
{% endhighlight %}

All done! :)
