---
layout: post
title: Install phpPgAdmin on Ubuntu 7.10
permalink: /2007/11/install-phppgadmin-on-ubuntu-710
tags:
- apache
- linux
- postgresql
- sql
- tutorial
- ubuntu
---

<p class="alert">
    <strong>Update:</strong> These instructions have been tested and work fine in the
    latest version of Ubuntu (8.04, Hardy Heron).
</p>

[phpPgAdmin](http://www.phppgadmin.org/) is a web based GUI for administrating a
[PostgreSQL](http://www.postgresql.org/) database server.

Here's some quick notes on getting it installed easily on Ubuntu 7.10...

In the terminal enter the following:

{% highlight bash %}
sudo apt-get install phppgadmin
{% endhighlight %}

This will set up and install all of the phpPgAdmin packages. It will also set-up and configure Apache and
php5 for you too if you haven't installed these already.

Next we need to create a symlink to phpPgAdmin so that Apache can find it:

{% highlight bash %}
sudo ln -s /etc/phppgadmin/apache.conf /etc/apache2/conf.d/phppgadmin.conf
{% endhighlight %}

Now if you navigate to [http://localhost/phppgadmin](http://localhost/phppgadmin) you should be greeted
with the phpPgAdmin screen. If your user account has a PostgreSQL account however, you will be logged in
automagically.

Optionally, if you would like to be able to use the phpPgAdmin interface as the default 'postgres'
administration account, (I am assuming here that you have set-up your PostgreSQL server using my
[set-up instructions](/2007/11/install-postgresql-on-ubuntu-710/) and therefore have a password
protected 'postgres' account and that logins require passwords.) you will need to do the following
(Please make sure you understand the security implications of allowing this type of access to your
database server - if you have not secured your administration accounts, do it now!)...

{% highlight bash %}
sudo gedit /usr/share/phppgadmin/conf/config.inc.php
{% endhighlight %}

Now find and change the following line

{% highlight php %}
$conf['extra_login_security'] = true;
{% endhighlight %}

to

{% highlight php %}
$conf['extra_login_security'] = false;
{% endhighlight %}

Save and close gedit.  Now all you need to do is restart Apache.

{% highlight bash %}
sudo /etc/init.d/apache2 reload
{% endhighlight %}

Now if you head on over to [http://localhost/phppgadmin](http://localhost/phppgadmin) all should be ready
for you.
