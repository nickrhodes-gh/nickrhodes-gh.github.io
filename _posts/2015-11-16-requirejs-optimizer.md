---
layout: post
title: No more optimisation headaches thanks to RequireJS
description: >
  A very quick look at how to use the RequireJS optimizer to combine and
  compress the projects JavaScript files into a single file.

---

After having some success with RequireJS as a dependency loader, I decided to
try my hand at getting the optimiser running as well. Basically all this does
is run through all of the dependencies that have been named in the current
project, then combine them into a single file and minify them. For some reason
I'm not using the grunt RequireJS task&hellip; ah well.

{% highlight js %}
npm install requirejs --save-dev
{% endhighlight %}

I decided to use a `build.js` file to store the optimizer config just to make
coming back to the config easier in the future. My JavaScript files are all
contained within a single `js` directory so this is where I stuck the
`build.js` file.

{% highlight js  %}
({
  baseUrl: ".",
  name: "app", // app.js
  out: "app-built.js", // output file
  paths: {
    jquery: "empty:" // unmap the jquery alias
  }
})
{% endhighlight %}

My build file is pretty simple, the only thing worth noting is the fact that I
need to essentially un-map the jQuery name. The optimizer is unable to
download network resources, so to prevent a nonexistent jQuery being combined
into the output file, this ensures the CDN URL stays intact.

You run the optimizer with `node <path to>/r.js -o <path to>/build.js`. The
modules which are combined are dictated by the require functions - the output
from `r.js` should show you what modules have been combined and compressed.

In my previous post [Getting Started with
RequireJS](/getting-started-with-requirejs/) I had the RequireJS script point
to `app.js`. However the optimiser spits out an `app-built.js` (due to my
config) file after the build process, so the script `src` needs to change in
the page.

{% highlight html %}
<script
  async
  data-main="../js/app-built"
  src="<link-to-requirejs">
</script>
{% endhighlight %}

Now I can continuously build the modules using `app-built.js` while rest of my
configuration remains exactly the same.

#### The RequireJS Grunt task

With me now using grunt tasks, it's a no-brainer to move all of this over to
`grunt-contrib-requirejs`, but that's for another day.
