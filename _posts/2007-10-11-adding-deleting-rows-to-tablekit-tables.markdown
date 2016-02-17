---
layout: post
title: Adding/Deleting Rows In TableKit Tables
tags:
- catalyst
- javascript
- perl
- prototype
- tablekit
- tutorial
---

<p class="alert">
    This post has now been <a href="/2007/11/adding-deleting-rows-in-tablekit-tables-revisited/">updated</a>.
    Please head over to the new post for something better...
</p>

Following on from my [previous post](/2007/09/making-editable-tables-with-catalyst-and-tablekit/) on
how to integrate the excellent [TableKit](http://www.millstream.com.au/view/code/tablekit/) into your
[Catalyst](http://www.catalystframework.org/) webapp (to make your data tables dynamically editable),
here's how i've gone about adding ajax inserts and deletes so that you can add and remove data rows in
your tables.

You'll need the code from the [previous post](/2007/09/making-editable-tables-with-catalyst-and-tablekit/)
to follow along.

We're going to be using prototype to do the background ajax calls, as this is already in our app as a
requirement of TableKit. Now let's get on with doctoring our template. First we add in a delete button
for each row of our table. Open up `/root/src/users/list.tt` and change the code of our table to look
like the following (the changes come on lines 9, 17, 20 and 26-28):

{% highlight html %}
<table id="users" class="sortable resizable editable">
  <thead>
    <tr>
      <th id="id" class="sortfirstasc noedit">Id</th>
      <th id="firstname">First Name</th>
      <th id="lastname">Last Name</th>
      <th class="noedit nocol"></th>
    </tr>
  </thead>
  <tfoot>
    <tr>
      <th>Id</th>
      <th>First Name</th>
      <th>Last Name</th>
      <th class="nocol"></th>
    </tr>
  </tfoot>
  <tbody id="user_body">
  [% FOREACH user IN users -%]
    <tr id='[% user.id %]'>
      <td>[% user.id %]</td>
      <td>[% user.firstname %]</td>
      <td>[% user.lastname %]</td>
      <td class="nocol">
        <a class="delete" href="#" onclick="userDelete([% user.id %]); return false">delete</a>
      </td>
    </tr>
  [% END -%]
  </tbody>
</table>
{% endhighlight %}

The above code adds a 'delete' link into each row of the table. This link is to run a short javascript
function (called `userDelete`) that we shall add now at the bottom of the template (within the original
javascript block):

{% highlight javascript %}
userDelete = function( user_id ) {
  var answer = confirm('Are you sure you want to delete this user?');
  if (answer) {
    new Ajax.Updater( 'user_body',
      '[% c.uri_for('/users/_delete_user/') %]?user_id=' + user_id,
      { asynchronous: 1, }
    );
    return false;
  }
}
{% endhighlight %}

This function basically asks the user to confirm that they want to delete the user entry, the sets up an
Ajax.Updater that passes the `user_id` to be deleted to a new function in our controller (that we shall
write next) which deletes it, and then passes the new table body back to be updated on screen. Let's now
create a new function `_delete_user` (and a helper function `__return_all_users`) in our user controller
`/lib/MyApp/Controller/Users.pm`:

{% highlight perl %}
=head _delete_user

Ajax method to delete users from the users table

=cut

sub _delete_user : Local {
  my ($self, $c) = @_;

  # Look-up our user entry
  my $user = $c->model('MyAppDB::Users')->find({
    id => $c->req->params->{user_id}
  })->delete();

  my $output = __return_all_users( $self, $c, 'null' );

  unless ( $output ) {
    $output = '<tr><td colspan="4" class="nocol">All Users Deleted</td></tr>';
  }

  $c->res->body( $output );
}

=head2 __return_all_users

Private method for the Users ajax interaction.
Returns the inner body of a html table.

=cut

sub __return_all_users : Private {
  my ( $self, $c, $new_user_id ) = @_;

  my @users = $c->model('MyAppDB::Users')->all(
    { order_by => 'id ASC' }
  );

  my $html;
  for ( my $i = 0 ; $i < scalar(@users) ; $i++ ) {
    my $class;

    if ( $new_user_id && $users[$i]->id == $new_user_id ) {
      $class = 'new';
    } else {
      if ( $i % 2 == 0 ) { $class = 'rowodd'; }
      else { $class = 'roweven'; }
    }

    $html .= "<tr class=\"" . $class . "\" id=\"" . $users[$i]->id . "\">
      <td>" . $users[$i]->id . "</td>
      <td>" . $users[$i]->firstname . "</td>
      <td>" . $users[$i]->lastname . "</td>
      <td class=\"nocol\">
        <a class=\"delete\" href=\"#\" onclick=\" userDelete("
        . $users[$i]->id . "); return false\">delete</a>
      </td>
      </tr>";
  }

  return ( $html );
}
{% endhighlight %}

The `_delete_user` function, takes the `user_id` that we sent to it, deletes the entry from the
database, then calls the `__return_all_users` function to rebuild the table body. This table body is
then used to update the table back on the web page.

That's the delete functionality sorted, now on with the row adding... Back in the template, add we add
another Ajax.Updater link below the table:

{% highlight html %}
<br />
<a class="add" href="#" onclick="
  new Ajax.Updater( 'user_body',
    '[% c.uri_for('/users/_add_user') %]',
    { asynchronous: 1, }
  ); return false ">add a user</a>
{% endhighlight %}

And add another new method into our controller to add a new user into the database and table:

{% highlight perl %}
=head _add_user

Ajax method to add users to the users table

=cut

sub _add_user : Local {
  my ($self, $c) = @_;

  # create a new user...
  my $new_user = $c->model('MyAppDB::Users')->create(
    {
      id        => $c->req->params->{user_id},
      firstname => '[First Name]',
      lastname  => '[Last Name]'
    }
  );

  my $output = __return_all_users( $self, $c, $new_user->id );

  $c->res->body( $output );
}
{% endhighlight %}

This function, like the delete function, does its business on the database, then rebuilds the table body
and sends it back to the web page.

The final icing on the cake, is a little addition to our CSS to make this all look a little nicer. Open
up `/root/css/myapp.css` and add the following to the bottom of the file:

{% highlight css %}
.nocol, th.nocol {
  background-color: #fff;
  border: none;
}

tr.new {
  background-color: #ff9697;
}

a {
  outline: none;
}

a.add, a.delete {
  background: url('/images/add.png') left center no-repeat;
  padding: 2px 0 2px 20px;
}

a.delete {
  background: url('/images/delete.png') left center no-repeat;
  left: 0;
}

tr a.delete {
  font-size: 0.8em;
}
{% endhighlight %}

And there you go. Now you can edit, add and delete users to your database and table without having to
reload the page.

<img src="/images/2007/add-remove-tablekit.png" alt="add-remove-tablekit.png" class="center border" />

However, there is one small bug that needs to be squashed to make this perfect... TableKit caches the
tables on page load - if you have a go at re-sorting one of the tables after deleting or adding a row,
this causes the table to essentially double in size as the original cached data is reloaded onto the
screen! :( I now need to figure out a way to clean out the cache for the current table and re-initalise
TableKit. Any suggestions would be more than welcome! :)

The files used in the examples can be downloaded below.

#### Attached Files:

<a href='/downloads/2007/myapp2.zip' title='MyApp - v2' class="archive">MyApp - v2</a>
