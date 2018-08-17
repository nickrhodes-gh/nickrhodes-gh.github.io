---
layout: post
title: >
    Dual booting Ubuntu and Fedora, while preserving the kernel command line
    parameters for each distro
description: >
    Preserve the custom GRUB_CMDLINE_LINUX parameters from your Fedora
    installation when updating grub config from Ubuntu

---

The background to this is that I've been running Ubuntu and Fedora alongside
each other for some time; one as my personal desktop and the other for
work. The only issue was that Fedora's Nvidia drivers would only load when using
the Fedora installed grub.

I have always made a note of installing the Nvidia drivers, and so far in Ubuntu
I have not had a single issue with them. This changed when I went went to
install them in the second Fedora session and found that they simply would not
load without installing grub from Fedora. Fortunately Ubuntu and the Nvidia
drivers didn't care which grub installation I was using, so the basic workaround
was to run with the Fedora installed grub.

In the end curiosity got the better of me and I decided to find out what was
going on.

Ubuntu seems to honour the blacklist directives within
`/etc/modprobe.d/nvidia-graphics-drivers.conf` and does not require any changes
to the `GRUB_CMDLINE_LINUX` parameter in `/etc/default/grub`.

```
$ cat /etc/modprobe.d/nvidia-graphics-drivers.conf
blacklist nouveau
blacklist lbm-nouveau
alias nouveau off
alias lbm-nouveau off
```

Whereas Fedora relies on the addition of several boot parameters to prevent the
kernel from loading the default nouveau drivers.

```
GRUB_CMDLINE_LINUX="rd.driver.blacklist=nouveau modprobe.blacklist=nouveau nvidia-drm.modeset=1"
```

After a quick look at `/boot/grub/grub.cfg` in Ubuntu it was clear that the
Fedora kernel boot parameters were missing, with just `root=/dev/sdb1` being
present (for reference these are the default parameters returned by
`/var/lib/linux-boot-probes/mounted/90fallback`).  It turns out the
`update-grub` in Ubuntu was not picking up Fedora's kernel parameters, and
appending them to the end of the `linux` line.

```
...
    linux /boot/vmlinuz-4.17.12-200.fc28.x86_64 root=/dev/sda2
...
```

My understanding of the `os-prober` script is that it works by scanning for
unmounted partitions and temporarily mounting them to then search for the
`/boot` directory. Once it finds a `/boot` directory it will run the grub.cfg
file through a series of 'test' files located in
`/usr/lib/linux-boot-probes/mounted/`. The scripts are looking for the menu
entry blocks, and specifically the `linux` and `initrd` lines, amongst others.


The issue with using the Ubuntu `os-prober` script is that it only searches for
`linux` and `initrd`, whereas the Fedora `grub.cfg` files contain `linux16` and
`initrd16` at the start of the lines, hence the `os-prober` fails to find the
correct Fedora menu entries.

In the end I copied `/usr/lib/linux-boot-probes/mounted/40grub2` to
`41grub2-linux16`, and replaced the `linux` and `initrd` case matches with
`linux16` and `initrd16` respectively. I then checked to see if the test command
`linux-boot-prober` printed out the correct kernel parameters for each kernel in
Fedora's `/boot`.

```patch
@@ -64,7 +64,7 @@
                                        ignore_item=1
                                fi
                        ;;
-                       linux)
+                       linux16)
                                # Hack alert: sed off any (hdn,n) but
                                # assume the kernel is on the same
                                # partition.
@@ -77,7 +77,7 @@
                                        kernel="/boot$kernel"
                                fi
                        ;;
-                       initrd)
+                       initrd16)
                                initrd="$(echo "$2" | sed 's/(.*)//')"
                                # Initrd same.
                                if [ "$partition" != "$bootpart" ]; then

```

After making the change, Ubuntu's `update-grub` command successfully found all of the kernel
arguments for Fedora, and my Nvidia drivers were working again with an Ubuntu
installed grub!
