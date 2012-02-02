---
layout: post
title: Visualizing your DBIC schema
permalink: /2007/10/visualizing-your-dbic-schema
tags:
- catalyst
- mysql
- oracle
- perl
- sql
linkblog: http://blog.jrock.us/articles/Visualizing%20your%20DBIC%20schema.pod
---

What a great idea - will give this a bash next week...

> If you want a somewhat pretty picture of your DBIC schema (with
> relationships drawn, of course), install GraphViz, SQL::Translator, and
> DBICx::Deploy from the CPAN, and then run:
>
> `$ dbicdeploy -Ilib MyApp::Schema ~/graphs GraphViz`
>
> ~/graphs will then contain a .sql file that is actually a png of your
> schema. Rename it and see your schema in your favorite png viewing
> application.
