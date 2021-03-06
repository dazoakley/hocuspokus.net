---
layout: post
title: Setting up SPF and DKIM for email on Bytemark Symbiosis
tags:
- email
- web-hosting
- bytemark
- spf
- dkim
---

---

**EDIT 17-February-2016:** Please DO NOT use these instructions to configure your
server for SPF and DKIM.  Bytemark Symbiosis now supports this out of the box - please
see the following documentation for more information:

[http://symbiosis.bytemark.co.uk/docs/symbiosis.html#s-spf-dkim](http://symbiosis.bytemark.co.uk/docs/symbiosis.html#s-spf-dkim)

---

I've been setting up a new [phpBB forum](http://www.phpbb.com) for my
[homebrew club](http://londonamateurbrewers.co.uk) (LAB) over the last week or so
and one of the main problems I found was that all the emails from the forum were
going straight to spam in email services such as Gmail. Not very useful when that's
one of the first forms of interaction with the user (activating their account via
email).

Thankfully there are ways to make your emails less spammy, (that I never knew about
until earlier this week) - [SPF](http://www.openspf.org/) (Sender Policy Framework)
for letting email recipients know which servers/domains are valid sources of email
for your domains; and [DKIM](http://www.ietf.org/rfc/rfc4871.txt) (DocumentKeys
Indentified Mail) for signing emails sent from your servers using public/private key
pairs. These to techniques combined should stop emails being sent to spam
automatically (It worked for me, but YMMV).

I run the LAB site on a [Bytemark](http://www.bytemark.co.uk) VM using their
[Symbiosis](http://symbiosis.bytemark.co.uk) server set-up.  This is basically Debian
Linux with a bunch of (automatic) scripts to make setting up new domains, DNS and
email as simple as possible - I really like this as it lets me not have to care too
much about this stuff. But, out of the box, the emails are not setup with SPF or
signed via DKIM - this is in their [feature backlog](https://projects.bytemark.co.uk/issues/4136),
but it's not in place yet. So... here are the quick pitted notes of how I set this up
for the domains on my server and the sites that I got the information from.

**NOTE:** This is for a single Bytemark Symbiosis VM running multiple sites handling
everything - DNS, email, web server etc. using the stock Symbiosis setup (I've not
modified any of the base system).

## SPF

The first (and most simple) thing to set-up was SPF as this is achieved by adding a
DNS entry that verifies where emails for this domain name can be sent from.

All this involved was adding the following line to the bottom of the tinydns
configuration file for my domain
(`~/londonamateurbrewers.co.uk/config/dns/londonamateurbrewers.co.uk.txt`):

{% highlight text %}
:londonamateurbrewers.co.uk:16:\016v=spf1\040mx\040-all:86400
{% endhighlight %}

This says that email is only allowed to be sent from the `mx.` subdomain - exactly
what I want. This line was generated by the SPF tool on [http://anders.com/projects/sysadmin/djbdnsRecordBuilder/#SPF](http://anders.com/projects/sysadmin/djbdnsRecordBuilder/#SPF)
by simply entering the following information:

* Domain: `londonamateurbrewers.co.uk`
* SPF Rules: `v=spf1 mx -all`
* Time to Live: `86400`

I then forced the DNS entry to be propagated to Bytemark's DNS servers with the
following command (or you could just wait 15 minutes for it to happen automatically):

{% highlight text %}
sudo /usr/sbin/symbiosis-dns-generate --upload
{% endhighlight %}

Finally, I used [http://tools.bevhost.com/spf/](http://tools.bevhost.com/spf/), to
verify that SPF was set-up correctly for the domain.

## DKIM

Next up was the slightly more complicated DKIM.  This involves DNS additions and
signing outgoing emails with public/private key pairs. I take no credit whatsoever
for solving this one - I knocked together the information from the various different
sources in the references section below.

So, first up, we need to generate the keys using openssl, move them to the right
location, and ensure they have the right permissions:

{% highlight text %}
cd $HOME
openssl genrsa -out rsa.private 1024
openssl rsa -in rsa.private -out rsa.public -pubout -outform PEM
sudo mv rsa.p* /etc/exim4/
cd /etc/exim4/
sudo chown Debian-exim:root rsa.p*
{% endhighlight %}

Now we have to copy the public key info into DNS.  This involves inserting the following
line into the tinydns configuration file, (below the SPF entry), and pasting in **YOUR**
public key just after the `p=`

{% highlight text %}
:key1._domainkey.londonamateurbrewers.co.uk:16:\352v=DKIM1;\040k=rsa;\040p=<INSERT-YOUR-PUBLIC-KEY-HERE>:86400
{% endhighlight %}

Again, I then forced a DNS update to take place:

{% highlight text %}
sudo /usr/sbin/symbiosis-dns-generate --upload
{% endhighlight %}

Finally, we have to tell Exim (the mail transport agent on Symbiosis servers) to sign emails
with this key...

First, create a file called `/etc/exim4/dkim_senders` and add in the following content (one
line for each domain you wish to send singned emails):

{% highlight text %}
*@londonamateurbrewers.co.uk: londonamateurbrewers.co.uk
{% endhighlight %}

Now to edit `/etc/exim4/symbiosis.d/20-routers/10-dnslookup` to look like this:

{% highlight conf %}
  # This router routes addresses that are not in local domains by doing a DNS
  # lookup on the domain name. The exclamation mark that appears in "domains = !
  # +local_domains" is a negating operator, that is, it can be read as "not". The
  # recipient's domain must not be one of those defined by "domainlist
  # local_domains" above for this router to be used.
  #
  # If the router is used, any domain that resolves to 0.0.0.0 or to a loopback
  # interface address (127.0.0.0/8) is treated as if it had no DNS entry. Note
  # that 0.0.0.0 is the same as 0.0.0.0/32, which is commonly treated as the
  # local host inside the network stack. It is not 0.0.0.0/0, the default route.
  # If the DNS lookup fails, no further routers are tried because of the no_more
  # setting, and consequently the address is unrouteable.

dnslookup_dkim:
  debug_print = "R: dnslookup_dkim for $local_part@$domain"
  driver = dnslookup
  domains = ! +local_domains
  senders = lsearch*@;/etc/exim4/dkim_senders
  transport = remote_smtp_dkim
  same_domain_copy_routing = yes
  # ignore private rfc1918 and APIPA addresses
  ignore_target_hosts = 0.0.0.0 : 127.0.0.0/8 : 192.168.0.0/16 :\
                        172.16.0.0/12 : 10.0.0.0/8 : 169.254.0.0/16 :\
                        255.255.255.255
  no_more

dnslookup:
  debug_print = "R: dnslookup for $local_part@$domain"
  driver = dnslookup
  domains = ! +local_domains
  transport = remote_smtp
  same_domain_copy_routing = yes
  # ignore private rfc1918 and APIPA addresses
  ignore_target_hosts = 0.0.0.0 : 127.0.0.0/8 : 192.168.0.0/16 :\
                        172.16.0.0/12 : 10.0.0.0/8 : 169.254.0.0/16 :\
                        255.255.255.255
  no_more
{% endhighlight %}

And then `/etc/exim4/symbiosis.d/30-transports/10-remote-smtp` to look like this:

{% highlight conf %}
# This transport is used for delivering messages over SMTP connections.

remote_smtp_dkim:
  debug_print = "T: remote_smtp_dkim for $local_part@$domain"
  driver = smtp
  dkim_domain = ${lookup{$sender_address}lsearch*@{/etc/exim4/dkim_senders}}
  dkim_selector = key1
  dkim_private_key = /etc/exim4/rsa.private
  dkim_canon = relaxed
  dkim_strict = false
  #dkim_sign_headers = DKIM_SIGN_HEADERS

remote_smtp:
  debug_print = "T: remote_smtp for $local_part@$domain"
  driver = smtp
{% endhighlight %}

Now run the following commands to copy these config changes across and restart Exim:

{% highlight text %}
cd /etc/exim4/
sudo make
sudo /etc/init.d/exim4 restart
{% endhighlight %}

That's should be about it, now all you have to do is test it out using
[this tool](http://www.appmaildev.com/en/dkim/).

## References

* Bytemark Symbiosis docs: [http://symbiosis.bytemark.co.uk/docs/symbiosis.html](http://symbiosis.bytemark.co.uk/docs/symbiosis.html)
* SPF
  * SPF Website: [http://www.openspf.org/](http://www.openspf.org/)
  * TinyDNS SPF Record Builder: [http://anders.com/projects/sysadmin/djbdnsRecordBuilder/#SPF](http://anders.com/projects/sysadmin/djbdnsRecordBuilder/#SPF)
  * SPF Test: [http://tools.bevhost.com/spf/](http://tools.bevhost.com/spf/)
* DKIM
  * DKIM Proposal: [http://www.ietf.org/rfc/rfc4871.txt](http://www.ietf.org/rfc/rfc4871.txt)
  * Using DKIM in Exim: [http://subhrajitnandy.wordpress.com/using-dkim-in-exim/](http://subhrajitnandy.wordpress.com/using-dkim-in-exim/)
  * Using DKIM in Exim: [http://www.debian-administration.org/users/lee/weblog/41](http://www.debian-administration.org/users/lee/weblog/41)
  * DKIM Test: [http://www.appmaildev.com/en/dkim/](http://www.appmaildev.com/en/dkim/)
