---
layout: post
title: More TUI fun with Ranger and Tig
description: >
  Some more text based program that run in your terminal

---

This is just a short follow on to my [last
post](2016/05/02/switching-to-mutt.html) which looked at setting up Mutt. This
post is about two new TUIs, Ranger and Tig, which provide a file explorer and
visual git revision graph respectively.

### Ranger

![Ranger screenshot]({{ site.url }}/assets/images/ranger.jpg)

Ranger is a text based file manager which uses Vim based keybindings to
navigate the file system. Initially it isn't as quick as moving using `cd
directory`, however it's got a well thought out layout and is easy to pick up.

It's pretty good at guessing what application to use when opening fies, and
for those occasions when it can't work it out it will prompt you for input.

### Tig

![Tig screenshot]({{ site.url }}/assets/images/tig.jpg)

I've been struggling to find a nice Git revision GUI for Linux for a little
while. There are several out there but they either have a Java dependency, or
aren't great to look at. I then discovered Tig which is a terminal based git
graph client.

Still pretty new to Tig so I'm still using most of the default configuration
aside from customising some of the colours. That's probably my only criticism
so far - the crazy number of highlight groups used in the default interface.
It makes quickly gathering useful info quite hard as you're presented with a
rainbow of text in the commit view. I've set quite a few of the highlight
groups to the same colour to make things a little less crazy.

{% highlight bash %}
##
# tigrc
##

# general colours
color   default                 15      default
color   cursor                  15      241
color   delimiter               213     default
color   author                  10      default
color   date                    12      default
color   line-number             221     default

# commit graph
color   main-tag                213     default     bold
color   main-local-tag          213     default
color   main-remote             11      default     bold
color   main-replace            81      default
color   main-tracked            221     default     bold
color   main-ref                81      default
color   main-head               12      default     bold
color   graph-commit            226     default

# diff   colors
color   diff_add                10      default
color   diff_add2               10      default
color   diff_del                9       default
color   diff_del2               red     default
color   diff-header             250     default
color   diff-index              250     default
color   diff-chunk              250     default
color   diff_oldmode            221     default
color   diff_newmode            221     default
color   'deleted file mode'     221     default
color   'copy from'             223     default
color   'copy to'               221     default
color   'rename from'           221     default
color   'rename to'             221     default
color   diff_similarity         250     default
color   'dissimilarity'         221     default
color   'diff_tree'             81      default
color   diff-stat               12      default
color   "Reported-by:"          156     default
color   'Author:'               250     default
color   'Commit:'               250     default
color   'AuthorDate:'           250     default
color   'CommitDate:'           250     default
color   'Date:'                 250     default
{% endhighlight %}

These are all personal preference, however it's definitely easier to parse the
diff with all the noise cancelled out.
