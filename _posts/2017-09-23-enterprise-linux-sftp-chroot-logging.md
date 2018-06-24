---
layout: post
title: >
  Enabling SFTP transaction logging in a chroot environment in Enterprise
  Linux 6.9
description: >
  By default sshd is unable to log SFTP transactions for users logging into a
  chroot environment. There are a couple of options for getting it running,
  this post explores the easiest.

---

**Quick note:** this is aimed at rsyslog version 5.8.10.

```sh
[root@centos6 ~]# rpm -qi rsyslog
Name        : rsyslog                      Relocations: (not relocatable)
Version     : 5.8.10                            Vendor: CentOS
Release     : 10.el6_6                      Build Date: Wed 17 Dec 2014
09:52:43 GMT
....
```

Setting up an SFTP server to jail users in a chroot environment will
considerably improve the security of the server by preventing the user from
having any awareness of the wider file system; as apposed to just relying on
permissions.

### Getting started

#### sshd

The most straight forward configuration involves adding the following to the
bottom of `/etc/ssh/sshd_config`.

```
# Set VERBOSE logging for debugging and facility LOCAL3 - change these
# as necessary.
Subsystem       sftp    internal-sftp # -l VERBOSE -f LOCAL3 # uncomment for none chroot user logging

# Example of overriding settings on a per-group basis
Match Group sftponly
        ChrootDirectory /data/%u
        X11Forwarding no
        AllowTcpForwarding no
# Enable logging for users in the chroot environment
        ForceCommand internal-sftp -l VERBOSE -f LOCAL3
```

This hands off control of the sftp server to `sshd` as there is no access to
the `sftp-server` binary from within chroot.

We also define a match group to apply some group specific rules to; it's this
group which will be our ftp_user's primary group.

#### User/file system

Assuming a test user called `ftp_user` - they should be assigned primary group
`sftponly` and ideally prevented from getting terminal access to the server.

```
$ id ftp_user
uid=501(ftp_user) gid=502(sftponly) groups=502(sftponly)

$ grep ftp_user /etc/passwd
ftp_user:x:501:502::/files:/sbin/nologin
```

The permissions on the `/data` directory are quite specific.

```
drwxr-xr-x. 3 root      root        4096 Sep 21 19:27 /data/
drwxr-xr-x. 4 root      sftponly    4096 Sep 21 19:51 /data/ftp_user/       <-- chroot directory
drwxr-xr-x. 2 root      root        4096 Sep 21 20:09 /data/ftp_user/dev/
drwxr-xr-x. 2 ftp_user  sftponly    4096 Sep 21 20:17 /data/ftp_user/files/ <-- users home directory
```

The `ftp_user` will log into the `/data/ftp_user` directory. From this
directory they have full access to the files directory (change the names as
necessary obviously).  The `/data/ftp_user/dev` directory is required for the
logging; it's in here that `rsyslog` will create the unix socket through
which sshd will log.

#### rsyslog

Really all that's left is to notify `rsyslog` of the unix socket on
`/data/ft_user/dev/log`. This is done by adding several lines to the end of
`/etc/rsyslog.conf`.

```
$InputUnixListenSocketCreatePath on
$AddUnixListenSocket /data/ftp_user/dev/log
local3.*                                                /var/log/sftp.log
```

The online documentation for `rsyslog-5.8.10` is awful to say the least. The
best place to look is `$ w3m /usr/share/doc/rsyslog-5.8.10/imuxsock.html` as
it contains some fairly well formatted examples at the end.

After restarting the `rsyslog` service there should be a unix socket file
called `log` in `/data/ftp_user/dev/`. You can test to see if `rsyslog` is
correctly listening on this socket by running `$ logger -d -u
/data/ftp_user/dev/log -p local3.info This is a log message`. This will the
corresponding log message to `/var/log/sftp.log`. Now when connecting into the
SFTP server as user `ftp_user`, sshd should log all SFTP transactions to the
`sftp.log`.

I haven't managed to get this running yet with SELinux in enforcing mode yet.
Currently when in enforcing mode users get permission denied when running `ls`
directories due to a context miss-match.

The final result is a log that tracks user login, directory listing, changing
of directories and file read/writes.

```
Sep 21 20:17:46 centos6 internal-sftp[5083]: session opened for local user ftp_user from [192.168.1.1]
Sep 21 20:17:46 centos6 internal-sftp[5083]: received client version 3
Sep 21 20:17:46 centos6 internal-sftp[5083]: realpath "."
Sep 21 20:17:50 centos6 internal-sftp[5083]: opendir "/"
Sep 21 20:17:50 centos6 internal-sftp[5083]: closedir "/"
Sep 21 20:17:52 centos6 internal-sftp[5083]: realpath "/files"
Sep 21 20:17:52 centos6 internal-sftp[5083]: stat name "/files"
Sep 21 20:17:56 centos6 internal-sftp[5083]: open "/files/testfile" flags WRITE,CREATE,TRUNCATE mode 0664
Sep 21 20:17:56 centos6 internal-sftp[5083]: close "/files/testfile" bytes read 0 written 815
Sep 21 20:17:59 centos6 internal-sftp[5083]: opendir "/files"
Sep 21 20:17:59 centos6 internal-sftp[5083]: closedir "/files"
Sep 21 20:18:02 centos6 internal-sftp[5083]: session closed for local user ftp_user from [192.168.1.1]
```

