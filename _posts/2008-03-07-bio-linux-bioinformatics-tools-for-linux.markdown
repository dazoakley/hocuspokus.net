---
layout: post
title: Bio-Linux - Bioinformatics Tools for Linux
permalink: /2008/03/bio-linux-bioinformatics-tools-for-linux
tags:
- bioinformatics
- howto
- linux
- ubuntu
---

[Bio-Linux](http://nebc.nox.ac.uk/biolinux.html) is a specialised Linux disro that provides both standard
and cutting edge bioinformatics software tools on a Linux base.

I wrote a post on my old blog a little while back now detailing how to use the packages from Bio-Linux in
[Ubuntu Linux](http://www.ubuntu.com/), but it got missed in the migration (sorry to all those who have
been searching for it). Here's a repost and update for Ubuntu 7.10...

Log into your system and open up a terminal, then follow these steps:

{% highlight bash %}
sudo gedit /etc/apt/sources.list
{% endhighlight %}

Add the following lines to the end of the file:

{% highlight plain_text %}
# Bio-Linux package repository
deb http://envgen.nox.ac.uk/bio-linux/ unstable bio-linux
{% endhighlight %}

Save and close the file, now back at the terminal type the following (Note that we need to install the
`bio-linux-staden` package, without this we'd have to do a bit more hacking in config files to stop
getting errors whenever we open up a terminal):

{% highlight bash %}
sudo apt-get update
sudo apt-get install bio-linux-base-directories bio-linux-staden
{% endhighlight %}

Next step is to set-up your environment for Bio-Linux...

{% highlight bash %}
sudo gedit /etc/bash.bashrc
{% endhighlight %}

Add the following lines to the end of the file:

{% highlight bash %}
# Set up Bio-Linux Environment
source /usr/local/bioinf/config_files/aliasrc
source /usr/local/bioinf/config_files/bioenvrc
{% endhighlight %}

Save and close the file.

That's it, now all of the packages available through Bio-Linux are now available to you in Ubuntu. One
final word of warning... I've noticed that a few of the packages are a little out of date, (the Bio-Linux
version of Taverna is 1.4, but 1.7 is [now available](http://taverna.sourceforge.net/)) so it might
be worth checking the version numbers before installing things.

The other thing to note with the Bio-Linux version of Taverna - it doesn't start properly without a
little bit of hacking... :( (If you decided to go with the Bio-Linux version, you can launch it by
running the following in the terminal: `/usr/local/bioinf/taverna/taverna/runme.sh`).
