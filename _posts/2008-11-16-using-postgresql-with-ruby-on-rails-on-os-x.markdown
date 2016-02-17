---
layout: post
title: Using PostgreSQL with Ruby on Rails on OS X
tags:
- postgresql
- ruby
- ruby-on-rails
- tutorial
---

Following on from my [last post](/2008/11/install-postgresql-on-mac-os-x-leopard/) for getting PostgreSQL
up and running nicely on Mac OS X, my next task was getting it playing nicely with ruby on rails - I'm off on
a rails course next week so I'm getting stuff ready. :)

It appears that there is currently two/three gem packages for using postgresql with ruby: `postgres`,
`ruby-pg`, and `pg` - and from what I can make out, they're all maintained by the same team now, (`postgres`
was the original package, but it got abandoned - `ruby-pg` and `pg` are the replacements). As such, i'll go
with the newer `ruby-pg` gem...

{% highlight bash %}
sudo env ARCHFLAGS="-arch i386" gem install \
ruby-pg -- \
--with-pgsql-lib=/opt/local/lib/postgresql83 \
--with-pgsql-include=/opt/local/include/postgresql83
{% endhighlight %}

Then finally, when writing the `database.yml` entry for connecting to postgresql, we have to define our adapter
as follows:

{% highlight yaml %}
development:
  adapter: postgresql
  database: test_dev
  encoding: unicode
  host: localhost
  user: XXXXX
  password: XXXXX
  timeout: 5000
{% endhighlight %}
