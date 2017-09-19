---
layout: post
title: Customising Tawk.to live chat
description: >
  A simple introduction to the Tawk.to API with examples showing how to
  customise the chat widget using the callbacks.

---

I spent hours searching for a live chat provider whose prices weren't
extortionate and client wasn't a complete eyesore. After several close calls I
stumbled upon [Tawk.to](https://www.tawk.to) - a 100% free live chat service
that also provides live user tracking and statistics.

After using it for a week I've found it offers some amazing features as well
as the live chat:

- Live user tracking in the chat window. The chat window will actually show
  live updates as the user navigates around the site.

- User page timers. The customers time spent on the site is logged, and from I
  can remember so is the time spent in individual pages.

- Conversations are saved. The same user can return later (not sure if there
  is an upper limit) and the previous conversation is still available.

- There are apps for Android, iOS and desktop!

So far it's been a fantastic service.

### Customising the chat client

Tawk.to provide some basic client configuration in their web-backend such as:
chat window colour, width and default text.

Luckily Tawk.to also provide an API through which you can hook into various
widget statuses. A very basic (but good enough for my needs) example showing
how to change additional widget styles.

{% highlight js %}
Tawk_API = Tawk_API || {};
Tawk_API.onLoad = function(){

  //  frame minimised
  var frameMin = $('#tawkchat-minified-iframe-element');
  var frameMinContents = frameMin.contents(); frameMinContents
    .find(".theme-background-color")
    .css("background-color","#6FBF98");

  frameMinContents
    .find("#tawkchat-minified-container")
    .css("border","none");

  frameMinContents
    .find("#tawkchat-minified-container")
    .css("border-width","0");

  //  frame maximised
  var frameMax = $('#tawkchat-maximized-iframe-element');
  var frameMaxContents = frameMax.contents();
  frameMaxContents
    .find("#popoutChat").hide();

  frameMaxContents
    .find("#headerBox.theme-background-color")
    .css("background-color","#6FBF98");

  frameMaxContents
    .find("#borderWrapper")
    .css("background-color","#6FBF98");

  frameMax.css('width',275);

};

// widget code...
{% endhighlight %}

Be careful though, watching visitors move around your site is surprisingly
addictive!
