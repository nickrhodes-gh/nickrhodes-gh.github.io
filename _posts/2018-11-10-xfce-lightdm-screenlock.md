---
layout: post
title: >
    Setting up screen locking in xfce4
description: >
    Getting screen locking to work in xfce4 with lightdm and light-locker

---

From an clean Arch install I was struggling to get lightdm playing nicely with
several screen locking applications. The symptoms were always the same
regardless of the locker installed:

- the initial login prompt on first boot would display and function without
  issue
- when running an of `xflock4`, `light-locker-command -l`, or `dm-tools lock`
  the screen would go black and would not return despite using the
  mouse/keyboard
- typing the password at the black screen would successfully unlock the desktop

It was also possible to display the lightdm login prompt by switching to another
vt (e.g. vt6 ctrl+alt+F6) and back to vt8.

In the end I was able to resolve the issue by installing the `xf86-video-intel`
package. After a reboot the display is now resuming to the lightdm login prompt
when using the mouse/keyboard.
