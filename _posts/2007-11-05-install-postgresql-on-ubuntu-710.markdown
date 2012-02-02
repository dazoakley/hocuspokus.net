---
layout: post
title: Install PostgreSQL on Ubuntu 7.10
permalink: /2007/11/install-postgresql-on-ubuntu-710
tags:
- geek
- linux
- postgresql
- tutorial
- ubuntu
---

<p class="alert">
    If you are using the latest version of Ubuntu (8.04 - Hardy Heron), you might find these
    <a href="/2008/05/install-postgresql-on-ubuntu-804/">slightly updated instructions</a> useful.
</p>

This quick walk-through are my notes for installing the PostgreSQL database server and the PgAdmin
administration application on Ubuntu Linux, and also set up the server so it allows access to other PC's
on your network.

Before we move on, this guide was tested on the current release of Ubuntu Linux, (7.10 - Gutsy Gibbon) and
PostgreSQL 8.2, but it should also be applicable to older versions (of Ubuntu and PostgreSQL) and other
Debian based distros.

Right for the basic installation, at the command-line, enter the following commands (or search for the
listed packages in synaptic if you prefer that way of working):

{% highlight text %}
sudo apt-get install postgresql postgresql-client postgresql-contrib
sudo apt-get install pgadmin3
{% endhighlight %}

This installs the database server/client, some extra utility scripts and the pgAdmin GUI application for
working with the database.

Now we need to reset the password for the 'postgres' admin account for the server, so we can use this for
all of the system administration tasks. Type the following at the command-line (substitute in the password
you want to use for your administrator account):

{% highlight text %}
sudo su postgres -c psql template1
template1=# ALTER USER postgres WITH PASSWORD 'password';
template1=# \q
{% endhighlight %}

That alters the password for within the database, now we need to do the same for the unix user 'postgres':

{% highlight text %}
sudo passwd -d postgres
sudo su postgres -c passwd
{% endhighlight %}

Now enter the same password that you used previously.

Then, from here on in we can use both pgAdmin and command-line access (as the postgres user) to run the
database server. But before you jump into pgAdmin we should set-up the PostgreSQL admin pack that enables
better logging and monitoring within pgAdmin. Run the following at the command-line:

{% highlight text %}
sudo su postgres -c psql < /usr/share/postgresql/8.2/contrib/adminpack.sql
{% endhighlight %}

Finally, we need to open up the server so that we can access and use it remotely - unless you only want to
access the database on the local machine. To do this, first, we need to edit the postgresql.conf file:

{% highlight text %}
sudo gedit /etc/postgresql/8.2/main/postgresql.conf
{% endhighlight %}

Now, to edit a couple of lines in the 'Connections and Authentication' section...

Change the line:

{% highlight text %}
#listen_addresses = 'localhost'
{% endhighlight %}

to

{% highlight text %}
listen_addresses = '*'
{% endhighlight %}

and also change the line:

{% highlight text %}
#password_encryption = on
{% endhighlight %}

to

{% highlight text %}
password_encryption = on
{% endhighlight %}

Then save the file and close gedit.

Now for the final step, we must define who can access the server. This is all done using the pg_hba.conf
file. (The following advice can also be given to you - plus you don't even need to figure out IP addresses
and subnet masks - from the latest versions of pgAdmin (1.6.x). However, this is not the version that ships
with Ubuntu, so i'll leave these instructions here.)

{% highlight text %}
sudo gedit /etc/postgresql/8.2/main/pg_hba.conf
{% endhighlight %}

Comment out, or delete the current contents of the file, then add this text to the bottom of the file:

{% highlight text %}
# DO NOT DISABLE!
# If you change this first entry you will need to make sure that the
# database
# super user can access the database using some other method.
# Noninteractive
# access to all databases is required during automatic maintenance
# (autovacuum, daily cronjob, replication, and similar tasks).
#
# Database administrative login by UNIX sockets
local   all         postgres                          ident sameuser
# TYPE  DATABASE    USER        CIDR-ADDRESS          METHOD

# "local" is for Unix domain socket connections only
local   all         all                               md5
# IPv4 local connections:
host    all         all         127.0.0.1/32          md5
# IPv6 local connections:
host    all         all         ::1/128               md5

# Connections for all PCs on the subnet
#
# TYPE DATABASE USER IP-ADDRESS IP-MASK METHOD
host    all         all         [ip address]          [subnet mask]  md5
{% endhighlight %}

and in the last line, add in your subnet mask (i.e. 255.255.255.0) and the IP address of the machine that
you would like to access your server (i.e. 138.250.192.115). However, if you would like to enable access
to a range of IP addresses, just substitute the last number for a zero and all machines within that range
will be allowed access (i.e. 138.250.192.0 would allow all machines with an IP address 138.250.192.x to
use the database server).

That's it, now all you have to do is restart the server:

{% highlight text %}
sudo /etc/init.d/postgresql-8.2 restart
{% endhighlight %}

And all should be working.
