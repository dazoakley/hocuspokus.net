---
layout: post
title: ! 'My First Gem: Biomart'
permalink: /2009/08/my-first-gem-biomart
tags:
- biomart
- gem
- ruby
---

A lot of my work these days involves building and querying biomart datasets. Interacting with
the webservices (for queries) is pretty simple, you just have to post a piece of xml at the
right url, but I found myself reusing a lot of boilerplate code in each and every new script.

So, to make my life a little easier (and hopefully some others) I packaged all of the code up
in a module with classes to handle all of the monkey work for me and decided to release it as a
gem. :)

The source code is available from [github](http://github.com/dazoakley/biomart/tree/master), 
and the documentation is on [rdoc.info](http://rdoc.info/projects/dazoakley/biomart).

For more details, check out the 
[release anouncement](http://rubyforge.org/forum/forum.php?forum_id=34404).

It's my first ruby module so any advice or pointers on how to make things better would be most
appreciated. :)
