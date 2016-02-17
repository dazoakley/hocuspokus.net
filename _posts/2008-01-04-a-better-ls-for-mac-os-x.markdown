---
layout: post
title: A better ls for Mac OS X
tags:
- colour-scheme
- command-line
- ego
- geek
- mac
- os-x
- terminal
- tutorial
---

I'm a bit of a command-line freak and like to spend a fair amount of time with the terminal open... As
such I like to spend a small amount of time getting the terminal set-up nicely. Other than changing the
default colour scheme and font, one (slightly) more drastic change is to replace the standard
implementation of `ls` for one that is slightly more configurable.

The default `ls` on OS X comes from BSD and compared to the GNU/Linux alternative is slightly lacking
when it comes to changing how things look - so what I like to do is replace it with the GNU `ls`
available in [MacPorts](http://www.macports.org/) - this allows me to get a terminal setup like below:

<img
  src="/images/2008/terminal.png"
  alt="terminal.png"
  title="terminal.png"
  class="center border" />

To get this done is pretty simple, once you have MacPorts set up correctly (if you can type `man port`
and get a manual page you're ready), just run the following command:

	sudo port install coreutils +with_default_names

This installs the 'GNU File, Shell, and Text utilities' which `ls` is part of - the extra option at the
end `+with_default_names` makes it override (only override - not replace, this is totally removable) the
default `ls` and other tools otherwise they will have a 'g' prefix - i.e. `ls` would be `gls`.

Next we add some extra configuration to our `~/.bash_profile` file (i'll include my MacPorts config in
case you get stuck above)...

{% highlight bash %}
# MacPorts
export PATH=/opt/local/bin:/opt/local/sbin:$PATH
export MANPATH=/opt/local/share/man:$MANPATH

# Terminal colours (after installing GNU coreutils)
NM="\[\033[0;38m\]" #means no background and white lines
HI="\[\033[0;37m\]" #change this for letter colors
HII="\[\033[0;31m\]" #change this for letter colors
SI="\[\033[0;33m\]" #this is for the current directory
IN="\[\033[0m\]"

export PS1="$NM[ $HI\u $HII\h $SI\w$NM ] $IN"

if [ "$TERM" != "dumb" ]; then
    export LS_OPTIONS='--color=auto'
    eval `dircolors ~/.dir_colors`
fi

# Useful aliases
alias ls='ls $LS_OPTIONS -hF'
alias ll='ls $LS_OPTIONS -lhF'
alias l='ls $LS_OPTIONS -lAhF'
alias cd..="cd .."
alias c="clear"
alias e="exit"
alias ssh="ssh -X"
alias ..="cd .."
{% endhighlight %}

Then finally we need to create a file called `.dir_colors` in our home directory that allows us to
configure the colours used by `ls`:

{% highlight bash %}
touch ~/.dir_colors
{% endhighlight %}

Then add the contents of the file here:

{% highlight text %}
# Configuration file for dircolors, a utility to help you set the
# LS_COLORS environment variable used by GNU ls with the --color option.

# The keywords COLOR, OPTIONS, and EIGHTBIT (honored by the
# slackware version of dircolors) are recognized but ignored.

# Below, there should be one TERM entry for each termtype that is colorizable
TERM linux
TERM linux-c
TERM mach-color
TERM console
TERM con132x25
TERM con132x30
TERM con132x43
TERM con132x60
TERM con80x25
TERM con80x28
TERM con80x30
TERM con80x43
TERM con80x50
TERM con80x60
TERM xterm
TERM xterm-color
TERM xterm-debian
TERM rxvt
TERM screen
TERM screen-w
TERM vt100

# Below are the color init strings for the basic file types. A color init
# string consists of one or more of the following numeric codes:
# Attribute codes:
# 00=none 01=bold 04=underscore 05=blink 07=reverse 08=concealed
# Text color codes:
# 30=black 31=red 32=green 33=yellow 34=blue 35=magenta 36=cyan 37=white
# Background color codes:
# 40=black 41=red 42=green 43=yellow 44=blue 45=magenta 46=cyan 47=white
NORMAL 00	# global default, although everything should be something.
FILE 00 	# normal file
DIR 01;36 	# directory
LINK 01;37 	# symbolic link.  (If you set this to 'target' instead of a
            # numerical value, the color is as for the file pointed to.)
FIFO 40;33	# pipe
SOCK 01;35	# socket
DOOR 01;35	# door
BLK 40;33;01	# block device driver
CHR 40;33;01 	# character device driver
ORPHAN 40;31;01 # symlink to nonexistent file

# This is for files with execute permission:
EXEC 01;35

# List any file extensions like '.gz' or '.tar' that you would like ls
# to colorize below. Put the extension, a space, and the color init string.
# (and any comments you want to add after a '#')

# If you use DOS-style suffixes, you may want to uncomment the following:
#.cmd 01;32 # executables (bright green)
#.exe 01;32
#.com 01;32
#.btm 01;32
#.bat 01;32

.tar 01;31 # archives or compressed (bright red)
.tgz 01;31
.arj 01;31
.taz 01;31
.lzh 01;31
.zip 01;31
.z   01;31
.Z   01;31
.gz  01;31
.bz2 01;31
.deb 01;31
.rpm 01;31
.jar 01;31
.dmg 01;31

# image formats
.jpg 01;35
.png 01;35
.gif 01;35
.bmp 01;35
.ppm 01;35
.tga 01;35
.xbm 01;35
.xpm 01;35
.tif 01;35
.png 01;35
.mpg 01;35
.avi 01;35
.fli 01;35
.gl 01;35
.dl 01;35

# source code files
.pl 00;33
.PL 00;33
.pm 00;33
.tt 00;33
.yml 00;33
.sql 00;33
.html 00;33
.css 00;33
.js 00;33
{% endhighlight %}

Finally, all you need to do is close and re-open the Terminal.  Now we should be sorted. :)
