---
layout: post
title: Dependency management with RequireJS
description: >
  Setting up RequireJS from a beginners point of view. This includes several
  examples of how to structure the modules.

---

On a whim I decided to see if it was possible to sequentially load JavaScript
modules based on their dependencies and it turns out it's not that hard. There
are several options out there, but in the end I went for
[RequireJS](http://requirejs.org/) as it seems to be the best maintained.

### RequireJS

To quote the [website](http://requirejs.org/):

> RequireJS is a JavaScript file and module loader. Using a modular script
> loader like RequireJS will improve the speed and quality of your code.

For my needs it's a dependency loader. If I have a module that requires
jQuery, all I have to do is list it as a dependency for my module. Once the
dependency has been created, Require will not load the module until jQuery has
loaded.  Obviously that's just the tip of the iceberg as to what can be done,
that's good enough for me.

#### Getting started

Firstly grab a copy of RequireJS
[here](http://requirejs.org/docs/download.html#requirejs). I've chosen to do
it using the [CloudFlare CDN](http://cdnjs.com/libraries/require.js/), however
RequireJS actually recommend downloading it locally. I've added `async` to to
the `script` tag as I don't care how it loads.

{% highlight html %}
<script
  async
  data-main="../js/app"
  src="https://cdnjs.cloudflare.com/ajax/libs/require.js/2.1.20/require.min.js">
</script>
{% endhighlight %}

The `data-main="../js/app"` defines your requirejs config file, you can also
use it as the entry file as well. My `app.js` file contains:

{% highlight js %}
// Setup require.js config
requirejs.config({
  "baseUrl": "../js",
  paths: {
    "app": "./",
    'jquery': '<jquery-url>'
  }
});

require(["main"]);
{% endhighlight %}

So I've included some things which technically I don't need: `"baseUrl":
"../js"` because all of my files are in the same directory as `../js/app`
which was specified in the `script src`, and `"app": "./"` for the same
reason. I wanted to keep these in here as I know I'll most likely change the
directory structure in the future.

Next there is the `'<jquery-url>'`. When you set a module dependency, you use
the filename without an extension, and this name is used to reference the
module in subsequent code. However because I'm loading jQuery from a CDN, it
can't automatically be given an alias name.  Hence why here we create a
mapping between the alias `'jquery'` and the content downloaded from
`'https://ajax.googleapis.com/ajax/libs/jquery/2.1.4/jquery.min'`.

The final part of the `app.js` file is the `require(["main"]);`. This states
that `main.js` should be run on start. Inside `main.js` is where the site wide
javascript functions are placed. If you're running a larger code base then
obviously you would break these out into separate files.

{% highlight js %}
//main.js
define(["jquery", "cookie"], function ($, cook) {

  // functions
  function monCookieControl() {
    var cookieSet = cook.getCookie('cookiecontrol');

    if ( cookieSet !== 'n' ) {
      html  = 'This website uses cookies. By continuing we assume'
      html += ' your permission to use cookies as detailed in the'
      html += ' <a href="/cookie-policy">cookie policy</a>'
      html += '<div id="closecookiecontrol">×</div>'

      $('#cookiecontrol').html(html)
      $('#cookiecontrol').css("padding", "15px 35px");

      $("#closecookiecontrol").click(function (e) {
        $('#cookiecontrol').html('');
        $('#cookiecontrol').css("padding", "5px 0");

        cook.setCookie('cookiecontrol', 'n');
      });
    }
  }

  monCookieControl()

});
{% endhighlight %}

Here I am using `define` to set which modules the enclosed script depends on.
Clearly it needs jQuery, and also a second module called cookie. I also create
two AMD return values, `$` and `cook` which allow me to reference `jquery` and
`cookie`.

The cookie module looks like.

{% highlight js %}
// cookies
define(function () {

  function setCookie(name, value, days) {
    if (days) {
      var date = new Date();
      date.setTime(date.getTime() + (days * 24 * 60 * 60 * 1000));
      var expires = "; expires=" + date.toGMTString();
    } else var expires = "";
    document.cookie = name + "=" + value + expires + "; path=/";
  }

  function getCookie(cname) {
    var name = cname + "=";
    var ca = document.cookie.split(';');
    for (var i = 0; i < ca.length; i++) {
      var c = ca[i];
      while (c.charAt(0) == ' ') c = c.substring(1);
      if (c.indexOf(name) == 0) {
        return c.substring(name.length, c.length);
      }
    }
    return "";
  }

  function delCookie(name, days) {
    if (days) {
      var date = new Date();
      date.setTime(date.getTime() + (days * 24 * 60 * 60 * 1000));
      var expires = "; expires=" + date.toGMTString();
    } else var expires = "";
    document.cookie = name + "=" + expires + "; path=/";
  }

  // return the functions for the require in main.js
  return {
    setCookie: setCookie,
    getCookie: getCookie,
    delCookie: delCookie
  }

})
{% endhighlight %}

In `cookie.js` `define` is used without any dependencies as it has none. It's
also important to return the functions we need to access in `main.js`.

There are probably a better ways to do what I've done in `monCookieControl()`,
but that's not the point of this post. Hopefully this simple quick start will
provide a base for someone else who it just discovering RequireJS.

##### HeadJS

Shout out to [HeadJS](http://headjs.com/). It actually offers a whole array of
features such as device/browser feature detection, view port queries and of
course JavaScript and CSS loading.

A basic example:

{% highlight js %}
// Load up some JS
head.load("jQuery.js", function() {
  // callback
  console.log("Done loading jQuery");
});

// Load up some CSS
head.load("bootstrap.css");
{% endhighlight %}

Great, that's super simple! My only issue with the module is that it hasn't
been updated [for a long time](https://github.com/headjs/headjs/issues/335).
As per the discussion in the link, I decided to try RequireJS.
