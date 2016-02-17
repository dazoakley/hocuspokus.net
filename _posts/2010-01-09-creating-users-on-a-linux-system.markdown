---
layout: post
title: Creating Users on a Linux System
tags:
- geek
- linux
- sudo
- user-management
---

It's been a while since I've had to do any basic sysadmin stuff on a Linux box... seeing as I
had to google the answers to this I thought I'd better write them down!

<strong>useradd syntax</strong>

{% highlight bash %}
useradd [options] {username}
{% endhighlight %}

Useful options are:

* `-d` - declare the users home directory
* `-m` - force 'useradd' to create the home directory
* `-D` - accept the system defaults for account settings

An alternative for 'useradd' though is 'adduser':

{% highlight bash %}
adduser {username}
{% endhighlight %}

This will then prompt you for all the information needed to set up the account.

<strong>making the user an administrator (with sudo)</strong>

As the root user...

{% highlight bash %}
visudo
{% endhighlight %}

Then underneath the entry for 'root' just add in an entry with the desired user name:

{% highlight bash %}
{username} ALL=(ALL) ALL
{% endhighlight %}
