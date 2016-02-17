---
layout: post
title: Remove Case-Sensitivity In Oracle
tags:
- oracle
- perl
---

Oracle is a great RDBMS, but the fact that searches against the database are case-sensitive can be a pain
in the butt. Here's how you can make searches case-insensitive...

In SQL...

{% highlight sql %}
alter session set NLS_SORT=BINARY_CI;
alter session set NLS_COMP=LINGUISTIC;
{% endhighlight %}

For use in Perl::DBI...

{% highlight perl %}
use strict;
use DBI;

my $dbh = DBI->connect(
  'dbi:Oracle:schema_name',
  'user',
  'password',
  {
    RaiseError    => 1,
    AutoCommit    => 0,
    on_connect_do => [
      'alter session set NLS_SORT=BINARY_CI',
      'alter session set NLS_COMP=LINGUISTIC'
    ]
  }
) || die "Database connection not made: $DBI::errstr";

$dbh->disconnect();
{% endhighlight %}
