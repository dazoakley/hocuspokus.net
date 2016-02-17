---
layout: post
title: Installing VMware Tools on Ubuntu Gutsy
tags:
- linux
- mac
- os-x
- ubuntu
- vmware
---

I've just started to play about with the new Ubuntu (7.10 - Gutsy), I have to say that I quite like it - I
was a big fan of Feisty and all other Ubuntu releases previously, so this is more of the same good stuff.

As I'm working with [VMWare Fusion](http://www.vmware.com/mac), the first thing that needs to be done is
install the VMWare Tools. This is quite simply done with the following lines in the terminal once you have
started your VM and mounted the tools image (Virtual Machine > Install VMWare Tools in the menu):

{% highlight bash %}
sudo aptitude update
sudo aptitude install build-essential linux-headers-$(uname -r)
cp -a /media/cdrom/VMwareTools* /tmp/
cd /tmp/
tar -vxzf VMwareTools*.gz
cd vmware-tools-distrib/
sudo ./vmware-install.pl
{% endhighlight %}

The just accept all of the defaults for the installation package.

Cheers to [Ubuntu Tutorials](http://ubuntu-tutorials.com/2007/10/02/how-to-install-vmware-tools-on-ubuntu-guests/)
for the info.
