---
layout: post
title: Mount ISO Files With Nautilus
permalink: /2007/11/mount-iso-files-with-nautilus
tags:
- geek
- gnome
- linux
- ubuntu
---

Here's two short bash scripts that allow you to mount and unmount an ISO image easily within the
right-click menu in Nautilus (the file manager in Gnome):

{% highlight bash %}
sudo mkdir /media/ISO
cd ~/.gnome2/nautilus-scripts/
gedit "Mount ISO Image"
{% endhighlight %}

Now paste this code into the file:

{% highlight bash %}
#!/bin/bash
#
for I in `echo $*`
do
foo=`gksudo -u root -k -m "enter your password for root terminal access" /bin/echo "got r00t?"`
sudo mount -o loop -t iso9660 $I /media/ISO
done
done
exit0
{% endhighlight %}

Save, and exit gedit.

{% highlight bash %}
gedit "Unmount ISO Image"
{% endhighlight %}

Then paste this code in this file:

{% highlight bash %}
#!/bin/bash
#
for I in `echo $*`
do
foo=`gksudo -u root -k -m "enter your password for root terminal access" /bin/echo "got r00t?"`
sudo umount $I
done
done
exit0
{% endhighlight %}

Save, and exit gedit. Now finally...

{% highlight bash %}
chmod a+x *
{% endhighlight %}

Now you can mount and unmount ISO files by using the 'Scripts' options in the right-click menu. :)

Thanks to the [Ubuntu Blog](http://ubuntu.wordpress.com/2005/10/24/nautilus-script-to-mount-iso-files/) for
the information.
