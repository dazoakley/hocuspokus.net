---
layout: post
title: Adding/Deleting Rows In TableKit Tables Revisited
tags:
- ajax
- catalyst
- geek
- javascript
- perl
- prototype
- tablekit
- tutorial
---

[TableKit](http://www.millstream.com.au/view/code/tablekit/) is a great javascript library for making
your HTML tables fully editable. However, one problem is that you can't add or delete rows from the
tables...

I came up with a solution to this
[not so long ago](/2007/10/adding-deleting-rows-to-tablekit-tables/), but it still had a problem
- TableKit caches the tables on loading, so after we update the table body (adding or deleting a row)
the sorting and editing of the table is completely screwed! :( However, with a little more work, and
some help from one of the guys in the office - we've finally got this cracked! :)

The basic idea to the fix is, instead of updating the table body, replace the entire table with a new
one and get TableKit to do its stuff on the new table. Although this is not the ideal solution (That
would be knowing more about javascript and the dom so that we can make changes to TableKits cache as we
update the table body - thus keeping the original table) - this _does_ work quite well. Here's the
details (We're using the code from the
[previous post](/2007/10/adding-deleting-rows-to-tablekit-tables/) as a starting point).

First, we'll update the code from our template file (`/root/src/users/list.tt`):

{% highlight html %}
[% META title = 'User List' -%]

<div id="users_div">
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
          <a class="delete" href="#" onclick="deleteUser([% user.id %]); return false">delete</a>
        </td>
      </tr>
    [% END -%]
    </tbody>
  </table>
</div>
<br />
<a class="add" href="#" onclick="addUser(); return false">add a user</a>
{% endhighlight %}

Then add this javascript code to the bottom of the same file:

{% highlight javascript %}
var users_table = new TableKit( 'users', {
  editAjaxURI: '[% c.uri_for('/users/_update_user') %]'
});

function addUser() {
  var timestamp = new Date().getTime();
  var new_table = 'users' + timestamp;
  var url = '[% c.uri_for('/users/_add_user') %]?timestamp=' + timestamp;
  new Ajax.Updater( 'users_div', url, {
    asynchronous: true,
    onComplete: function() {
    new TableKit( new_table, {
      editAjaxURI: '[% c.uri_for('/users/_update_user') %]'
    })
    }
  });
}

function deleteUser( user_id ) {
  var timestamp = new Date().getTime();
  var new_table = 'users' + timestamp;
  var url = '[% c.uri_for('/users/_delete_user/') %]?user_id=' + user_id + '&timestamp=' + timestamp;
  var answer = confirm('Are you sure you want to delete this user?');
  if (answer) {
    new Ajax.Updater( 'users_div', url, {
      asynchronous: true,
      onComplete: function() {
        new TableKit( new_table, {
          editAjaxURI: '[% c.uri_for('/users/_update_user') %]'
        })
      }
    });
  }
}
{% endhighlight %}

The main differences in the above from last time round are as follows:

*   The 'users' table is now inside a div with the id 'users_div'
*   The link to add a user now calls a javascript function rather than an Ajax.Updater call directly.
*   The functions to add and delete users are essentially the same:
*   They create a timestamp to individually identify the new table we are going to create (this is __the__ most important thing here to 		remember)!
*   They then use an Ajax.Updater call to talk to the Perl controller and generate the new table with a row added or removed.
*   Once the Ajax.Updater is complete, TableKit is then called again to make our new table editable and sortable.

That's the template sorted, now let's look at the controller (`/lib/MyApp/Controller/Users.pm`):

{% highlight perl %}
package MyApp::Controller::Users;

use strict;
use warnings;
use base 'Catalyst::Controller';

=head1 NAME

MyApp::Controller::Users - Catalyst Controller

=head1 DESCRIPTION

Catalyst Controller.

=head1 METHODS

=cut

=head2 index

=cut

sub index : Private {
  my ( $self, $c ) = @_;
  $c->response->redirect('/users/list');
}

=head2 list

Fetch all user objects and pass to users/list.tt in stash to be displayed

=cut

sub list : Local {
  my ( $self, $c ) = @_;
  $c->stash->{users}    = [ $c->model('MyAppDB::Users')->all ];
  $c->stash->{template} = 'users/list.tt';
}

=head2 _update_user

Ajax method to update the users table

=cut

sub _update_user : Local {
  my ( $self, $c ) = @_;

  $c->model('MyAppDB::Users')->find( { id => $c->req->params->{id} } )
    ->update( { $c->req->params->{field} => $c->req->params->{value} } );

  $c->res->body( $c->req->params->{value} );
}

=head2 _delete_user

Ajax method to delete users from the users table

=cut

sub _delete_user : Local {
  my ( $self, $c ) = @_;

  # Look-up our user entry
  my $user =
    $c->model('MyAppDB::Users')->find( { id => $c->req->params->{user_id} } )
    ->delete();

  my $output =
    __return_all_users( $self, $c, $c->req->params->{timestamp}, 'null' );

  $c->res->body($output);
}

=head2 _add_user

Ajax method to add users to the users table

=cut

sub _add_user : Local {
  my ( $self, $c ) = @_;

  # create a new user...
  my $new_user = $c->model('MyAppDB::Users')->create(
    {
      id        => $c->req->params->{user_id},
      firstname => '[First Name]',
      lastname  => '[Last Name]'
    }
  );

  my $output = __return_all_users( $self, $c, $c->req->params->{timestamp}, $new_user->id );

  $c->res->body($output);
}

=head2 __return_all_users

Private method for the Users ajax interaction.
Returns a html table.

=cut

sub __return_all_users : Private {
  my ( $self, $c, $timestamp, $new_user_id ) = @_;

  my @users = $c->model('MyAppDB::Users')->all();

  my $html =
      '<table id="users'
    . $timestamp
    . '" class="sortable resizable editable">
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
    <tbody id="user_body">';

  if ( scalar(@users) > 0 ) {
    for ( my $i = 0 ; $i < scalar(@users) ; $i++ ) {
      my $class;

      if ( $new_user_id && $users[$i]->id == $new_user_id ) {
        $class = 'new';
      }
      else {
        if   ( $i % 2 == 0 ) { $class = 'rowodd'; }
        else                 { $class = 'roweven'; }
      }

      $html .=
          "<tr class=\""
        . $class
        . "\" id=\""
        . $users[$i]->id . "\">
          <td>" . $users[$i]->id . "</td>
          <td>" . $users[$i]->firstname . "</td>
          <td>" . $users[$i]->lastname . "</td>
          <td class=\"nocol\">
              <a class=\"delete\" href=\"#\" onclick=\" deleteUser("
        . $users[$i]->id . "); return false\">delete</a>
          </td>
        </tr>";
    }
  }
  else {
    $html .=
      '<tr><td colspan="4" class="nocol">All Users Deleted</td></tr>';
  }

  $html .= '</tbody></table>';

  return ($html);
}

=head1 AUTHOR

Darren Oakley

=head1 LICENSE

This library is free software, you can redistribute it and/or modify
it under the same terms as Perl itself.

=cut

1;
{% endhighlight %}

The only real changes in the controller are a general code clean-up, and a slight change to the
'__return_all_users' method - this now returns a whole table (identified by the unique timestamped id
that we generated in the javascript functions). Nothing really needed changing in the back-end.

There you go! Now our TableKit based tables can be used to not only edit entries in the database, we can
now add and remove them! :)

The files used in this example can be found below.

#### Attached Files:

<a href="/downloads/2007/myapp3.zip" title="myapp.zip" title="MyApp - v3" class="archive">MyApp.zip - v3</a>
