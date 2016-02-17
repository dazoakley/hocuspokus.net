---
layout: post
title: Install PostgreSQL on Mac OS X Leopard
tags:
- datbase
- mac
- postgresql
- tutorial
---

This type of guide is all over the internet, but I'm too lazy to search for one every time I want to do this.
;) So here's a brief overview of how I got PostgreSQL set-up nicely on Mac OS 10.5...

<h3>Setting Up Our Environment</h3>

Using a text editor of your choice, add the following lines to the bottom of the `/etc/profile` file (you'll
need to be an administrator to do this):

{% highlight bash %}
# MacPorts
export PATH=/opt/local/bin:/opt/local/sbin:$PATH

# PostgreSQL
export PATH=/opt/local/lib/postgresql83/bin:$PATH
{% endhighlight %}

Now before we move on, make sure that you have [MacPorts](http://www.macports.org/) installed.

<h3>Installing PostgreSQL with MacPorts</h3>

I like MacPorts - it makes installing general Unix packages and keeping them up-to-date a snap. As such,
we'll use it to do the heavy lifting for installing PostgreSQL. Open up a terminal and hit up the following
command:

{% highlight bash %}
sudo port install postgresql83 postgresql83-server
{% endhighlight %}

Sit back and wait a while...  That's the basic install done.

Now to configure it, do the following:

{% highlight bash %}
sudo mkdir -p /opt/local/var/db/postgresql83/defaultdb
sudo chown postgres:postgres /opt/local/var/db/postgresql83/defaultdb
sudo su postgres -c '/opt/local/lib/postgresql83/bin/initdb -D /opt/local/var/db/postgresql83/defaultdb'
{% endhighlight %}

This installs a default database instance for us to be getting on with. Next, run this command to get
PostgreSQL to start on system boot:

{% highlight bash %}
sudo launchctl load -w /Library/LaunchDaemons/org.macports.postgresql83-server.plist
{% endhighlight %}

Reboot your Mac.

<h3>Creating a User Account and Database</h3>

Once your Mac is back up, we finally need to create an account for us to use (substitute 'daz' for your
username)...

{% highlight bash %}
createuser --superuser daz -U postgres
{% endhighlight %}

Now a database to play with...

{% highlight bash %}
createdb test_db
{% endhighlight %}

<h3>Admin Your Database with pgAdmin 3</h3>

Download [pgAdmin](http://www.pgadmin.org/), then run the following commands to set up some functions that
allows pgAdmin to collect statistics about your databases:

{% highlight bash %}
cd /opt/local/share/postgresql83/contrib/
sudo su postgres -c psql < adminpack.sql
{% endhighlight %}

That's it - job done!

For notes on making the database accessible over the network, please refer to one of my posts on
[PostgreSQL on Ubuntu](/2008/05/install-postgresql-on-ubuntu-804/).
