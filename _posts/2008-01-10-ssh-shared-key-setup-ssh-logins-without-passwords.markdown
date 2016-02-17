---
layout: post
title: SSH Shared-Key Setup - SSH Logins Without Passwords
tags:
- command-line
- geek
- linux
- mac
- os-x
- terminal
- tutorial
---

SSH is a great tool for remotely accessing another machine, but entering your password every time you log
into a remote box can be a pain if you would like to set-up some background scripts to connect to a
server and do something (i.e. a backup script running as a cron job). Here's how I set-up my Mac to be
able to log into my server without the need for a password to be entered each time - the instructions
should be good for any variant of Unix/Linux, but you need to take into account path names etc. on your
machine.

The first thing we will do is generate a key for the SHH version 1 protocol (just in case you are
connecting to an older machine):

{% highlight bash %}
ssh-keygen -t rsa1
{% endhighlight %}

SSH-Keygen will respond with something like the following:

{% highlight text %}
Generating public/private rsa1 key pair.
Enter file in which to save the key (/Users/daz/.ssh/identity):
{% endhighlight %}

At this point hit enter then you will be prompted for a passphrase - this is a form of password that will
be used to generate your unique keys and can contain any set of characters and spaces - something like
_&quot;I'm really liking all of this geeky nonsense!&quot;_ is a perfectly acceptable passphrase - just
whatever you do, don't use an empty passphrase. After entering (and confirming) your passphrase you will
get the following output:

{% highlight text %}
Your identification has been saved in /Users/daz/.ssh/identity.
Your public key has been saved in /Users/daz/.ssh/identity.pub.
{% endhighlight %}

This means that our identity keys have been generated. Now we just need to create a pair of keys for the
SSH2 protocol - you can use the same or different passphrases for these keys - it's up to you...

{% highlight bash %}
ssh-keygen -t dsa
{% endhighlight %}

Then

{% highlight bash %}
ssh-keygen -t rsa
{% endhighlight %}

You should now have three sets of keys in your `~/.ssh` directory, the ones with the .pub extension are
your public keys (what we need to put on your other machines) and the others are your private keys -
these must be kept safe!

So, let's use `scp` to copy the files across:

{% highlight bash %}
scp ~/.ssh/*.pub daz@MyServer:/home/daz/
{% endhighlight %}

Then log into your server using `ssh` and issue the following commands:

{% highlight bash %}
cat identity.pub >>~/.ssh/authorized_keys
cat id_dsa.pub >>~/.ssh/authorized_keys
cat id_rsa.pub >>~/.ssh/authorized_keys
rm identity.pub id_dsa.pub id_rsa.pub
{% endhighlight %}

This populates the `authorized_keys` file on our server with the three public keys that we have just
transferred and then removes them as they're no longer needed here.

That's everything done, now all we have to do is log out of our server, and then try and log back in via
ssh - a password should no longer be required!<sup>*</sup> :)

<sup>*</sup> This isn't strictly true - on OS X it asks you for the `id_rsa` passphrase that we
established before, you will need to enter this, but you can then have it stored in the keychain for
hassle free use from here.
