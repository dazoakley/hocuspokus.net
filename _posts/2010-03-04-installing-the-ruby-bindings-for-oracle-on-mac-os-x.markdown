---
layout: post
title: Installing the Ruby Bindings for Oracle on Mac OS X
tags:
- gem
- install
- mac
- oracle
- ruby
---

This quick crib sheet is for the guys at work (and anyone else who might need help on this
one)...

First install the [oracle instant client](http://www.oracle.com/technology/software/tech/oci/instantclient/index.html)
somewhere - I normally install it to `/usr/local/oracle/instantclient_10_2`. Then...

{% highlight bash %}
cd /usr/local/oracle/instantclient_10_2
ln -s libclntsh.dylib.10.1 libclntsh.dylib
{% endhighlight %}

Following this, run `mkdir -p /usr/local/oracle/network/admin/`, and place a copy of the
tnsnames.ora file in this directory.

Now you need to add the following to your `.profile` or `.bashrc` file:

{% highlight bash %}
# Oracle env variables
export ORACLE_HOME="/usr/local/oracle/instantclient_10_2"
export DYLD_LIBRARY_PATH="/usr/local/oracle/instantclient_10_2:$DYLD_LIBRARY_PATH"
export SQLPATH="/usr/local/oracle/instantclient_10_2"
export TNS_ADMIN="/usr/local/oracle/network/admin"
export NLS_LANG="AMERICAN_AMERICA.UTF8"
export PATH="$PATH:$DYLD_LIBRARY_PATH"
{% endhighlight %}

That's all of the set-up done, now you just need to install the gem:

{% highlight bash %}
gem install ruby-oci8
{% endhighlight %}

Done. :)
