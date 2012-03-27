---
layout: post
title: Copy/Paste Highlighted Code with Vim
permalink: /2012/03/copy-paste-highlighted-code-with-vim
tags: [vim,keynote,presentation]
---

So, I'm preparing a presentation to give at work for one of our "brown bag" /
"learning lunch" sessions and I want to put in some code into my slides...

When I used to use TextMate, I used the very cool [copy as RTF
bundle](https://github.com/drnic/copy-as-rtf-tmbundle), for such tasks.

But now I'm a Vim head, here's how to do the same in Vim...

First, install the
[highlight](http://www.andre-simon.de/doku/highlight/en/highlight.html) library
(`brew install highlight` on the mac), then install the [rtf-highlight Vim
plugin](https://github.com/dharanasoft/rtf-highlight)<sup>\*</sup> - I
recommend [vundle](https://github.com/gmarik/vundle) to manage your plugins.

Now all you have to do is run the command `:RTFHighlight <language>` and the
contents of your current buffer (or text selection) will be added to your
clipboard as glorious syntax highlighted rich text, ready to paste into
keynote etc! :)

<sup>\*</sup> You may need to apply the changes from this
[pull request](https://github.com/dharanasoft/rtf-highlight/pull/1) to the Vim
plugin to get it to work with more recent versions of the highlight library.

