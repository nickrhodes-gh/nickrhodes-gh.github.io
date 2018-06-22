---
layout: post
title: Storing Mutt email password in Gnome Keyring
description: >
  This post shows you have to store your email login password in Gnome Keyring
  instead of saving it directly into the muttrc file in plain text. You will be
  prompted for your keyring password the first time you start mutt, but after
  that it will log straight in.

---

It's not sensible to store passwords in plain text files, however I also don't
enjoy having to enter it every time I want to read my email. I chose to store
my password in the gnome-keyring, and script it's retrieval when Mutt is
started.  Gnome-keyring is a secure password/certificate store which only
requires a single password (usually the users login password) to unlock.

Surprisingly (as far as I'm aware) there are no programs in the default Linux
Mint repositories that will give you simple access to the keyring. I had to
download [gkeyring](https://github.com/kparal/gkeyring), a third party tool
written in Python, to query the keyring. The documentation for gkeyring says
it all so I'll not list everything here.

Some things worth noting&hellip; if you have auto-login enabled then your
default keyring is not unlocked when you log in. Instead you'll be prompted to
enter the keyring password the first time you try to access it.

Not all `gkeyring` arguments are the same. I was originally using `--id` to
query the keyring item, however doing it that way would not trigger the
password prompt when doing it for the first time.

```
gkeyring --id <id> -1
```

Instead I had to use `--name`, which for some reason will always prompt.

```
gkeyring --name <item_name> -1
```

Moving onto the Mutt side. I spent ages trying to work out why my
`$email_pass` variable wasn't working in my `muttrc`.

```
###
# .muttrc
###

set email_pass=`python gkeyring.py --name <item_name> -1`

# smtp
set smtp_pass = $email_pass
...

# imap
set imap_pass = $email_pass
...
```

It's actually required that you prepend all variables with `my_` for them to
work.

```
###
# .muttrc
###

set my_pass=`python gkeyring.py --name <item_name> -1`

# smtp
set smtp_pass = $my_pass
...

# imap
set imap_pass = $my_pass
...
```

Job done. Now when you open Mutt, you'll be able to log into your email after
entering your keyring password.
