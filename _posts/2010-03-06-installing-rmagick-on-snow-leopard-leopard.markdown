---
layout: post
title: Installing RMagick on Snow Leopard / Leopard
tags:
- gem
- install
- leopard
- mac
- os-x
- ruby
- snow-leopard
---

This used to be quite a complicated task to install ImageMagick and RMagick from source, but
these days (thanks to some great work by others) it's a piece of cake...

First install ImageMagick using this install script hosted on github
([http://github.com/masterkain/ImageMagick-sl](http://github.com/masterkain/ImageMagick-sl)):

{% highlight bash %}
cd ~/src
git clone git://github.com/masterkain/ImageMagick-sl.git
cd ImageMagick-sl
sh install_im.sh
{% endhighlight %}

Now simply sit back and wait. (Oh, you'll need to enter your password at some points as the
libraries are installed into `/usr/local`).

Then, once that's done, RMagick is installed by:

{% highlight bash %}
gem install rmagick
{% endhighlight %}

Simples. :)
