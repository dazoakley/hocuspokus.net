---
layout: post
title: Use Leopards Quick Look from the Terminal
permalink: /2007/11/use-leopards-quick-look-from-the-terminal
tags:
- mac
- os-x
linkblog: http://www.tuaw.com/2007/11/05/terminal-tip-use-quick-look-from-the-leopard-command-line/
---

Useful info from over at the Unofficial Apple Weblog:

> TUAW reader Shaun Haber sent us a link to his personal blog with a great post about using Leopard's Quick
> Look from the command line, which is wonderfully handy for anyone who spends a chunk of their day in
> Terminal. The `qlmanage` utility gives you direct access to many Quick Look functions; of specific interest
> is the -p flag. This option displays the Quick Look generated preview for any file. So if you tell it to
> `qlmanage -p foo.png`, the image immediately pops up in a Quick Look pane.
>
> Even better, Quick Look supports slide shows. So if you `cd` into a folder of images and run `qlmanage -p
> *.jpg`, you'll be rewarded with a full-on presentation of your pictures.
>
> Other qlmanage flags of interest include -h (displays a help message) -t (thumbnail generation) and -f (a
> zoom factor to display with).
>
> The downside of qlmanage is that it's full of NSLog-style messages. Haber recommends you pipe the output
> into /dev/null as follows: `qlmanage -p *.jpg >& /dev/null`.
