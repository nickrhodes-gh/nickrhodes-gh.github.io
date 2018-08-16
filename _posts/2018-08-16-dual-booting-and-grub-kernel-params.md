---
layout: post
title: >
    Dual booting Ubuntu and Fedora and preserving the kernel command line
    parameters for each distro
description: >
    Preserve the custom GRUB_CMDLINE_LINUX parameters from your Fedora
    installation when updating grub config from Ubuntu

---

The background to this is that I've been running Ubuntu and Fedora alongside
each other for some time. One's my personal desktop and the other I use for
work. I noticed soon after installing the Nvidia drivers in Fedora that they
were not being loaded after installing grub from the Ubuntu session, while they
worked fine when grub was installed from Fedora. Fortunately Ubuntu didn't
care which grub installation I was using which made pinpointing the issue much
easier.

Ubuntu seems to honour the blacklist directives within
`/etc/modprobe.d/nvidia-graphics-drivers.conf` and does not require any changes
to `GRUB_CMDLINE_LINUX` in `/etc/default/grub`, whereas Fedora relies on the
addition of several parameters to prevent the kernel from loading the nouveau
drivers.

```
GRUB_CMDLINE_LINUX="rd.driver.blacklist=nouveau modprobe.blacklist=nouveau nvidia-drm.modeset=1"
```

After a quick look at `/boot/grub/grub.cfg` in Ubuntu it was clear that the
Fedora kernel boot parameters were missing, with just `root=/dev/sdb1` being
present.  It turns out the `update-grub` in Ubuntu was not picking up Fedora's
kernel parameters. My understanding of the `os-prober` script is that it works
by scanning for unmounted partitions and temporarily mounting them to search
for the `/boot` directory. Once it finds a `/boot` directory it will then search
for and parse the `grub.cfg` file - it does this by looping through the scripts
in `/usr/lib/linux-boot-probes/mounted/`. The scripts are looking for the menu
entry sections, and specifically the `linux` and `initrd` lines, amongst others.


The issue with using the Ubuntu `os-prober` script is that it only searches for
`linux` and `initrd`, whereas the Fedora `grub.cfg` files contain `linux16` and
`initrd16`, hence the `os-prober` fails to find the Fedora menu entries.

In the end I edited `/usr/lib/linux-boot-probes/mounted/40grub2` by appended `16` to the
`linux` and `initrd` case matches, and saving the file as `41grub2-linux16`.
After this the Ubuntu `update-grub` command successfully found all of the kernel
arguments for Fedora, and my Nvidia drivers were working again.
