---
layout: post
title: Making Editable Tables with Catalyst and TableKit
permalink: /2007/09/making-editable-tables-with-catalyst-and-tablekit
tags:
- ajax
- catalyst
- javascript
- perl
- prototype
- tablekit
- tutorial
---

[Catalyst](http://www.catalystframework.org/) is an MVC web-development framework for Perl, very similar
in concepts, ideas (and even some of the implementation) to [Ruby on Rails](http://rubyonrails.org/).
I'm using Catalyst a lot in my job now, and one of the first challenges that i've had to go through is
making dynamically editable tables for some of the views we are using (in other words, Ajax driven
edit-in-place tables).

Following a short Google, there seemed to be only two ready-made options I could see: one, using the
Catalyst controller module
[Catalyst::Controller::ROSE::EIP](http://search.cpan.org/~karman/Catalyst-Controller-Rose-0.04/lib/Catalyst/Controller/Rose/EIP.pm);
or two, by integrating and using the stand-alone javascript package
[TableKit](http://www.millstream.com.au/view/code/tablekit/).

Unfortunately the first option, ROSE::EIP is dependent on the use of the
[Rose::DB::Object](http://search.cpan.org/~jsiracusa/Rose-DB-Object-0.765/lib/Rose/DB/Object.pm) as your
database ORM mapper. As our webapp is already using
[DBIx::Class](http://search.cpan.org/~ash/DBIx-Class-0.08007/lib/DBIx/Class.pm) as its ORM layer, that
rules ROSE::EIP out and TableKit was the way to go. Here's a quick rundown of how I got it going, using
a very simple table to store a users first and last names as an example (all files used in this tutorial
are available for download at the bottom of this post) and a single controller called 'users'.

First off, head on over to the [TableKit](http://www.millstream.com.au/view/code/tablekit/) site and
download the latest TableKit package, then create the following three directories in your `/MyApp/root/`
directory: *javascript*, *css*, and *images*.

From the TableKit download, copy the files: *tablekit.js*, *fastinit.js* and *prototype.js* (found in
scriptaculous/lib) into your new '*javascript*' directory; then copy the two images - *down.gif* and
*up.gif* into the *images* directory.

Now create a new file and add in the following CSS code:

{% highlight css %}
table {
  border-collapse: collapse;
}

td, th {
  padding: 0.5em;
  border: 1px solid #CCC;
}

thead, tfoot {
  background-color: #DDD;
}

thead {
  border-bottom: 2px solid #f08900;
}

tfoot {
  border-top: 2px solid #f08900;
}

tr.rowodd {
  background-color: #FFF;
}

tr.roweven {
  background-color: #F2F2F2;
}

.sortcol {
  cursor: pointer;
  padding-right: 20px;
  background-repeat: no-repeat;
  background-position: right center;
}

.sortasc {
  background-color: #DDFFAC;
  background-image: url('/images/up.gif');
}

.sortdesc {
  background-color: #B9DDFF;
  background-image: url('/images/down.gif');
}

.nosort {
  cursor: default;
}

th.resize-handle-active {
  cursor: e-resize;
}

div.resize-handle {
  cursor: e-resize;
  width: 2px;
  border-right: 1px dashed #1E90FF;
  position:absolute;
  top:0;
  left:0;
}
{% endhighlight %}

Save this file as '*myapp.css*' in your new *css* directory. This will give your tables a on okay
looking starting point and nice striping on odd and even rows when we're finished.

Next, open up the `/MyApp/root/lib/site/html` file and add the following lines within the head tags:

{% highlight html %}
<link href="/css/myapp.css" rel="stylesheet" media="screen" type="text/css" />
<script src="/javascript/prototype.js" type="text/javascript"></script>
<script src="/javascript/fastinit.js" type="text/javascript"></script>
<script src="/javascript/tablekit.js" type="text/javascript"></script>
{% endhighlight %}

Now, we update our `/MyApp/root/src/users/list.tt` file so that the users table looks like the following
snippet of code:

{% highlight html %}
[% META title = 'User List' -%]
<table id="users" class="sortable resizable editable">
  <thead>
    <tr>
      <th id="id" class="sortfirstasc noedit">Id</th>
      <th id="firstname">First Name</th>
      <th id="lastname">Last Name</th>
    </tr>
  </thead>
  <tfoot>
    <tr>
      <th>Id</th>
      <th>First Name</th>
      <th>Last Name</th>
    </tr>
  </tfoot>
  <tbody>
    [% FOREACH user IN users -%]
    <tr id="[% user.id %]">
      <td>[% user.id %]</td>
      <td>[% user.firstname %]</td>
      <td>[% user.lastname %]</td>
    </tr>
    [% END -%]
  </tbody>
</table>
{% endhighlight %}

In the above lines of code there are two important details that need to be done in order for the table
sorting and editing to work:

*One: The id properties*

The id properties on each column of the table are set to the same names as the fields in the database
table. and the table rows, (in the body of the table) also have an id associated with them - this
becomes the database row 'ID' attribute when the template is interpreted. These id's then become some of
the parameters that are passed via TableKit to our Ajax.Updater controller method.

*Two: The class properties*

In the table header, the id column has two classes applied to it - 'sortfirstasc' and 'noedit' - this
tells TableKit to first sort the table based on this column (and ascending), and that the values of this
column are not to be editable.

Also in the declaration of the table, the table has the classes: 'sortable' 'resizable' and 'editable' -
these tell TableKit what you'd like to be able to do with your table.

Finally, add the following snippet of Javascript code to the bottom of the same file
(`/MyApp/root/src/users/list.tt`):

{% highlight javascript %}
var comments_table = new TableKit( 'users', {
  editAjaxURI: '[% update_uri %]'
});
{% endhighlight %}

This initializes the TableKit actions and tells TableKit the uri that the Prototype Ajax.Updater is to
call (a Catalyst controller method that we're about to create).

The final step is now back in the 'Users' controller. Here's the code that we need for both the initial
list of users and the Ajax.Updater method that TableKit calls:

{% highlight perl %}
sub list : Local {
  my ($self, $c) = @_;
  $c->stash->{users} = [$c->model('MyAppDB::Users')->all];
  $c->stash->{template} = 'users/list.tt';
  $c->stash->{update_uri} = $c->uri_for('/users/_update_user');
}

sub _update_user : Local {
  my ($self, $c) = @_;

  $c->model('MyAppDB::Users')
    ->find({ id => $c->req->params->{id} })
    ->update({
      $c->req->params->{field} => $c->req->params->{value}
    });

  $c->res->body( $c->req->params->{value} );
}
{% endhighlight %}

There you go, you should now have a table on your '/users/list' page that is sortable, resizable, and
now even in-place editable. I hope these instructions are clear enough (there's quite a lot in there!),
any questions, comments or improvements people could give me would be most appreciated.

Next up, i'll go through how I made the tables more functional by adding in add and delete buttons (to
add and delete rows to the table, and therefore create new database entries). :)

#### Attached Files:

<a href='/downloads/2007/myapp.zip' title='MyApp' class="archive">MyApp</a>
