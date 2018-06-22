---
layout: post
title: Opencart database deployment tools (or lack of)
description: >
  Thoughts on why Opencart lacks any sort of database migration functionality,
  and how I've setup my migration process.

---

Since getting involved with Opencart I amazed at how many site owners
regularly work directly on their production database. Whether that's editing
stock lists in the backend to full on web dev. It's just too easy to mess up a
price or make a spelling mistake&hellip; the later isn't the end of the world,
but it definitely doesn't leave a good impression.

However the situation is not helped by the lack of acceptance deployment
tools.  As far as I can see, the options are: develop locally, then move the
data up either manually or even better, dump the necessary tables and import
them on the remote server. But even that's hard work, why not just have a
module that can do it all for you?

Amazingly I've been using the second option for the better half of two years.
I'm a constant bottle neck in the movement of content as only I have remote
access to the server (and I only have finite amount of time). Recently I found
myself with some spare time though, so decided to hack together a module to
take care of the transfer for me. It still needs to be run manually, although
anyone with permission can now do it.

### Opencart

The Opencart DB isn't complicated, however it's not so obvious which tables
contain the product/site data and tables which contain data specific to the
server (the ones you don't want to overwrite). Having been through the DB
several times, these are the tables I think are core tables that hold data
unique to production.

```
-- Tables holding core store data
oc_affiliate*
oc_customer*
oc_order*
oc_return*
oc_review*
oc_setting*
oc_user*
oc_voucher*
oc_coupon*
```

Using mysqldump it's pretty straight forward to ignore these.

```
# source DB
mysqldump --ignore-table <table_name> db_name > dump_file.sql

# dest DB
mysql db_name < dump_file.sql
```

But that all involves connecting into the server via ssh blah blah blah.

In an attempt to automate this slightly I decided to pipe the dump over ssh to
the destination server.

```
ignore= \
  $(mysql source_db -e "show tables" | \
  egrep "(customer|review|history|return|order|address|setting|user|affiliate)" | \
  sed -E "s/(.*)/--ignore-table source_db.\1/g" | tr "\n" " ")

if mysqldump \
   -u${MYSQL_USERNAME} \
   -p${MYSQL_PASSWORD} \
   -P${MYSQL_PORT} \
   -h${MYSQL_HOST} $ignore $source_db | \
   ssh $dest_server \
   'mysql \
     -u${MYSQL_USERNAME} \
     -p${MYSQL_PASSWORD} \
     -P${MYSQL_PORT} \
     -h${MYSQL_HOST} $dest_db'; \
 then exit 0; \
 else exit 1; \ #hopefully catch connection issues etc
 fi
```

This works great if you've setup your `~/.ssh/config` file to login using
certs.  You still need to ssh into the server to run the script. You could
schedule it with cron, although that seems like a sledgehammer approach.

The next step is to get this running in php. I'm pretty lazy, so the thought
of porting mysqldump into php doesn't appeal. Instead I decided to have php
run a shell script containing the mysqldump code above.  This means I can take
advantage of the Openshift environment variables which store the mysql login
details.

```
exec('path/to/script.sh', $output, $status_code);
```

The `if` statement in the shell script provides a return status to the php
script. It can be improved.

All that's left now is to get a basic module up and
running so the script can be triggered by an admin in the Opencart backend.
