---
layout: post
title: Default Text Field Values That Disappear on Focus
tags:
- javascript
- jquery
- prototype
---

You will have noticed this effect out on your travels on the internet - a text input field has some
default text in it (often in a slightly dimmed colour), and when you click on the text box, default text
disappears and then reappears when you click away from the box. It happens on the search box to the right
here too.

I needed this same effect at work today so a quick Google came up with this great
[blog post](http://webdeveloper.beforeseven.com/jquery/default-text-field-value-disappears-focus) - a
lovely example of the exact effect I was looking for in both raw javascript and a version using the
[jQuery](http://jquery.com/) library.

> The script searches the page that it's inserted on for all form input fields that have a class of
> 'default-value' applied. Each form input field must also have a unique ID.
>
> When the page loads the script changes the color of the text in the text fields it has found to the value
> of 'inactive\_color'. If the user clicks on the input field, the default text is blanked, and the colour
> changed to 'active\_color'. If the user clicks away from the input field, i.e. the input field loses
> focus, the value of the text field will revert back to the original text, and the colour will change back
> to 'inactive\_color', unless the user has entered some other text.

Here's a copy of the jQuery version for my own personal notes, never know when that could be useful:

{% highlight javascript %}
/*
 * Written by Rob Schmitt, The Web Developer's Blog
 * http://webdeveloper.beforeseven.com/
 */

var active_color = '#000'; // Colour of user provided text
var inactive_color = '#999'; // Colour of default text

$(document).ready(function() {
  $("input.default-value").css("color", inactive_color);
  var default_values = new Array();
  $("input.default-value").focus(function() {
    if (!default_values[this.id]) {
      default_values[this.id] = this.value;
    }
    if (this.value == default_values[this.id]) {
      this.value = '';
      this.style.color = active_color;
    }
    $(this).blur(function() {
      if (this.value == '') {
        this.style.color = inactive_color;
        this.value = default_values[this.id];
      }
    });
  });
});
{% endhighlight %}

However, we don't use jQuery at work, our entire site is based around
[Prototype](http://www.prototypejs.org/) and [script.aculo.us](http://script.aculo.us/), so I did a bit
of butchering and here's the code adapted to use Prototype:

{% highlight javascript %}
/*
 * Written by Darren Oakley
 * http://hocuspokus.net
 */

var active_color = '#000'; // Colour of user provided text
var inactive_color = '#999'; // Colour of default text

Event.observe( window, 'load', function () {
    var default_values = new Array();
    $$("input.default-value").each( function (s) {
        $(s).setStyle({ color: inactive_color });
        $(s).observe( 'focus', function () {
            if (!default_values[s.id]) {
                default_values[s.id] = s.value;
            }
            if (s.value == default_values[s.id]) {
                s.value = '';
                $(s).setStyle({ color: active_color });
            }
            $(s).observe( 'blur', function () {
                if (s.value == '') {
                    $(s).setStyle({ color: inactive_color });
                    s.value = default_values[s.id];
                }
            });
        });
    });
});
{% endhighlight %}

Now all you need to do is give your input text fields a unique id, and the class of 'default-value', then
the script will take care of the rest.
