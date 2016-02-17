---
layout: post
title: Regexp Find and Replace in Ruby's gsub
tags:
- ruby
- regexp
---

So I found this little trick quite useful the yesterday...

If you want to use a regular expression matched variable to find and replace
sections of text in a string more intelligently (I used to do this all the
time in Perl, hence why I was looking for a Ruby equivalent) you can!

Here's a rather contrived example but it gets the message accross.

{% highlight ruby %}
text = '<a href="#" class="button">click me</a><span class="weee">weee</span>'

puts text
# => <a href="#" class="button">click me</a><span class="weee">weee</span>

text.gsub!(/class="(.+)"/,'data-class="\1"')

puts text
# => <a href="#" data-class="button">click me</a><span data-class="weee">weee</span>
{% endhighlight %}


The Perl equivalent to `\1` would be the `$1` matched variable...
