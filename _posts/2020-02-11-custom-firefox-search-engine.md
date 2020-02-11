---
layout: post
title: >
    Creating custom search engines in Firefox Quantum
description: >
    A quick look at how to define and install your own custom search provider
    without resorting to installing additional addons.
---

The goal was to add a new search provider to the One-Click Search Engines list
in `about:preferences#search`. It seems the only way to do this is to either
install an existing addon from `addons.mozilla.org`, or use a provider that
supports the [OpenSearch description
format](https://developer.mozilla.org/en-US/docs/Web/OpenSearch#Autodiscovery_of_search_plugins).

I was trying to install a custom search for Linux man pages; there are quite a
few sites that host them online, all with a search option, so I figured it
shouldn't be too difficult. Firstly, there are no addons specifically for man
page search, and none of the hosted sites support OpenSearch apparently.

## Setup
- Create an OpenSearch XML document describing the search provider
- Create a single index.html page which links to the XML doc
- Serve the page locally!

### OpenSearch document
This is pretty well documented on the [Mozilla Developer
site](https://developer.mozilla.org/en-US/docs/Web/OpenSearch). I chose to use
http://man.he.net as my provider, which resutled in the following XML file.

```xml
<OpenSearchDescription xmlns="http://a9.com/-/spec/opensearch/1.1/"
                       xmlns:moz="http://www.mozilla.org/2006/browser/search/">
  <ShortName>Man Pages</ShortName>
  <Description>Man page search</Description>
  <InputEncoding>UTF-8</InputEncoding>
  <Image width="16" height="16">data:image/png;base64,iVBORw0KGg...SuQmCC</Image>
  <Url type="text/html" template="http://man.he.net/">
    <Param name="topic" value="{searchTerms}"/>
    <Param name="section" value="all"/>
  </Url>
  <moz:SearchForm>http://man.he.net/</moz:SearchForm>
</OpenSearchDescription>
```

It's all pretty self-explanatory; I've used a base64 encoded image for the
small icon that appears next to the provider in the One-Click Search Engines
list. The `<Url>` secion is where we actually define the search path, and the
parameters to pass to the search. `{searchTerms}` is a variable that is
replaced by your search term, and in my example, `name="section"` just notes
that we are searching all man page sections. Save this as `man-page.xml`.

The above results in the query URL http://man.he.net/?topic=whoami&section=all.

### Installing the custom search
This is pretty straight forward if you are able to spin up a local webserver
(just opening the html file in Firefox wasn't working for me).

Fristly you need to switch your location bar from the single location+search
box, to the separate location and search setup.

![firefox-search](/assets/images/firefox-search.png)

Then create a small `index.html` file in your currect directory. It only needs to
contain the following.

```htmlmixed=
<html>
    <head>
        <link rel="search"
              type="application/opensearchdescription+xml"
              title="Man Pages"
              href="/man-page.xml">
    </head>
</html>
```

After saving, run `python -m http.server 8000`, and visit http://0.0.0.0:8080
in your browser. You should see a small green `+` inside the search box to the
right of the location bar. Clicking this will install the new search provider.

Now you can assign a keyword, such as `@man`, so that when you type this into
the search bar, it will automatically search only the man page provider.
