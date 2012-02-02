---
layout: post
title: Add Git and SVN Branch to Bash Prompt
permalink: /2009/07/add-git-and-svn-branch-to-bash-prompt
tags:
- bash
- command-line
- geek
- git
- linux
- mac
- os-x
- svn
---

**EDIT:** (12-Mar-2010) slight update to the svn prompt code to give a better / more useful
output...

I've seen things like this posted on the net before but never really had a chance to play with
the idea. But as I'm now using git and svn a lot more these days (fingers crossed i'll be
totally free of cvs soon!) I thought it was about time I pulled my finger out.

So here's the end goal, in a normal directory, we just get a normal bash promt, but in a
directory controlled by git or svn, we also get an addition telling us the source control tool
in use and the current branch:

<img
  src="/images/2009/git_svn_bash_terminal.png"
  alt="git_svn_bash_terminal"
  title="git_svn_bash_terminal"
  width="500"
  class="center" />

So, fire up yer terminal and add the following to your `.profile`, `.bash_profile` or `.bashrc`
(whichever one you use):

{% highlight bash %}
parse_git_branch () {
  git name-rev HEAD 2> /dev/null | sed 's#HEAD\ \(.*\)# (git::\1)#'
}
parse_svn_branch() {
  parse_svn_url | sed -e 's#^'"$(parse_svn_repository_root)"'##g' | awk '{print " (svn::"$1")" }'
}
parse_svn_url() {
  svn info 2>/dev/null | sed -ne 's#^URL: ##p'
}
parse_svn_repository_root() {
  svn info 2>/dev/null | sed -ne 's#^Repository Root: ##p'
}

BLACK="\[\033[0;38m\]"
RED="\[\033[0;31m\]"
RED_BOLD="\[\033[01;31m\]"
BLUE="\[\033[01;34m\]"
GREEN="\[\033[0;32m\]"

export PS1="$BLACK[ \u@$RED\h $GREEN\w$RED_BOLD\$(parse_git_branch)\$(parse_svn_branch)$BLACK ] "
{% endhighlight %}

Simples. Now just open up a new terminal and move into a project directory using svn or git. :)
