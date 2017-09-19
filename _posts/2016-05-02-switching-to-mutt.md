---
layout: post
title: Getting Mutt up and running
description: >
  This post is really just an quick overview of my move from Thunderbird to
  mutt for managing my email.

---

Recently I've been reading more and more blogs by people who do almost
everything in their terminal which has inspired me to do the same. Some of the
common TUIs used are [Irssi](https://irssi.org/), [Rainbow
Stream](http://www.rainbowstream.org/) and my latest venture
[Mutt](http://www.mutt.org/).

On Linux Mint the installation is pretty trivial.

{% highlight bash %}
sudo apt-get update && sudo apt-get install mutt
{% endhighlight %}

You then need to setup the `~/.muttrc` config file. From what I've seen so
far, there's an endless list of options, however only a handful are needed to
actually connect to your mailbox.

{% highlight bash %}
###
# .muttrc
###

# smtp
set smtp_url = "smtp[s]://<username>@<host>[:port]/"
set smtp_pass = "<password>"
set from = "<your-email>"
set realname = "John Doe"


# imap
set imap_user = "<imap-username>"
set imap_pass = "<imap-password>"
set spoolfile = "imap[s]://<imap-host>/INBOX"
set folder = "imap[s]://<imap-host>/INBOX"
set record = "imap[s]://<imap-host>/INBOX.Sent"

# cache
set header = "~/.mutt/hcache"
set message_cachedir = "~/.mutt/mcache/"
set certificate_file = "~/.mutt/certificates"

# sensible defaults
set sort=reverse-threads
set sort_aux=last-date-received
set editor = "vim -c'setlocal spell spelllang = en_gb wrap linebreak nolist tw=0 wm=0 | map j gj | map k gk'"

# key bindings
bind index k  previous-entry
bind index j  next-entry
bind index K  previous-page
bind index J  next-page

bind pager k  previous-line
bind pager j  next-line
bind pager l  next-undeleted
bind pager h  previous-undeleted
bind pager ^h display-toggle-weed

# colours
# default for transparent background
color status      blue      default
color normal      white     default
color hdrdefault  cyan      default
color indicator   black     cyan
color markers     brightred default
color quoted      green     default
color signature   cyan      default
color tilde       blue      default
color tree        red       default
{% endhighlight %}

If you are using STARTTLS or SSL/TLS then you should use `imaps/smpts` in the
URLs. The colours are copied from `/etc/Muttrc.d/colors.rc` with some small
adjustments to swap the background colours.

Getting the mail index was straight forward. Sending mail was another story.
Initially I was being asked to sign into my account every time I tried to send
an email, only for it to hang on the `Logging In` status. In the end I tracked
this down to an incorrect `record` URL. Mutt debug mode is extremely useful
should you hit any issues during the setup.

{% highlight bash %}
mutt -d2 # 5 levels of debug 1-5
{% endhighlight %}

This starts Mutt logging to `~/.muttdebug0`. When Mutt first connects to your
mailbox, it'll request a listing of all folders.

{% highlight bash %}
4< * LIST (\Noselect) "." ""
4< a0002 OK LIST completed
4< * LSUB (\HasNoChildren) "." "INBOX.Junk"
4< * LSUB (\HasNoChildren) "." "INBOX.Deleted Messages"
4< * LSUB (\HasNoChildren) "." "INBOX.Sent Messages"
4< * LSUB (\HasNoChildren) "." "INBOX.Trash"
4< * LSUB (\HasNoChildren) "." "INBOX.spambucket"
4< * LSUB (\HasNoChildren) "." "INBOX.Sent"
4< * LSUB (\HasNoChildren) "." "INBOX.Drafts"
4< * LSUB (\HasNoChildren) "." "INBOX.Templates"
4< * LSUB (\Noselect \HasChildren) "." "INBOX"
4< a0003 OK LSUB completed
{% endhighlight %}

You can see how the sub-folders are listed by the mail server &mdash; this is
how you need to reference sub-folders in `.muttrc` file.

The key bindings are how I would expect Mutt to behave after coming from a Vim
background, with navigation being done by the `hjkl` keys.

The [Mutt manual](http://www.mutt.org/doc/manual/) is a good place to start.
I'd skip straight to the [index](http://www.mutt.org/doc/manual/#intro-index)
section if you are just starting out. However some of the index status are
below.

{% highlight bash %}
# Message status flags

D       message is deleted (is marked for deletion)
d       message has attachments marked for deletion
K       contains a PGP public key
N       message is new
O       message is old
P       message is PGP encrypted
r       message has been replied to
S       message is signed, and the signature is successfully verified
s       message is signed
!       message is flagged
*       message is tagged
n       thread contains new messages (only if collapsed)
o       thread contains old messages (only if collapsed)

# Message recipient flags

+       message is to you and you only
T       message is to you, but also to or CC'ed to others
C       message is CC'ed to you
F       message is from you
L       message is sent to a subscribed mailing list
{% endhighlight %}
