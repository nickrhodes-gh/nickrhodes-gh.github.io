---
layout: post
title: >
    Running a Wayland session in Gnome3 and Arch Linux
description: >
    How to enable starting a Wayland session by default with GDM in Arch Linux
    and the Gnome3 desktop environment.

---

First things first, I'm running a Dell XPS 9360, so my issues weren't related to
90% of the "GDM won't load wayland" threads which almost exclusively involve
Nvidia graphics cards.

It actually took me some time to realise that I wasn't running within a Wayland
session on my recent Arch install - I had just assumed that GDM would default to
Wayland over the fallback X11 session. However after hooking up a couple of
monitors over a thunderbolt connection, I noticed differences between my running
in Fedora and Arch installations.

A quick check revealed that I was in fact running the X11 session:

```sh
% echo $XDG_SESSION_TYPE
X11
```

Sifting through the journal logs, the only issues I could see were a missing
orca.desktop file (who cares), and gdm-x-session/gnome-shell complaining about
not being able to access the kms device.

```
Nov 24 21:33:00 arch /usr/lib/gdm-x-session[646]: (II) modesetting: Driver for Modesetting Kernel Drivers: kms
Nov 24 21:35:40 arch gnome-shell[384]: Failed to create backend: Could not find a primary drm kms device
```

It took a lot of searching, however I found [the
following](https://wiki.archlinux.org/index.php/Kernel_mode_setting#Early_KMS_start)
information regarding kms not starting early enough for GDM. The pretty straight
forward workaround is to add i915 to the `MODULES=()` list in
`/etc/mkinitcpio.conf`. After the reboot you need to rebuild the initramfs with
`mkinitcpio -p linux`, and reboot. You should be offered the choice of "Gnome",
"Gnome-classic" and "Gnome on X" sessions at the login prompt, and be able to
start running wayland.

```sh
% echo $XDG_SESSION_TYPE
wayland
```

