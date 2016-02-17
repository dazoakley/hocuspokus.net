---
layout: post
title: Creating New Accounts in PostgreSQL
tags:
- linux
- postgresql
- tutorial
---

Getting a new account set up on PostgreSQL is a simple process...

Create our new user:

{% highlight bash %}
sudo su postgres -c createuser daz
{% endhighlight %}

Then you have to give this new user role a name (I called it daz), and then say 'y' to the question
"Shall the new role be a superuser?" if you want the user to be an administrator.

Give the user a database password (this does not have to be the same as their unix password):

{% highlight text %}
sudo su postgres -c psql
postgres=# ALTER USER daz WITH PASSWORD 'mypassword';
postgres=# \q
{% endhighlight %}

Finally, give the new user a database to play with:

{% highlight bash %}
sudo su postgres -c createdb daz
{% endhighlight %}

Pretty straight forward... :)
