---
layout: post
title: >
    Firefox light page content with dark GTK themes
description: >
    Forcing Firefox to use a light theme for the page content even when using a
    dark GTK theme

---

Anyone who has used a dark GTK theme will have noticed that Firefox will render
any un-styled input boxes using the default dark GTK background. This often
makes for unreadable text within the input.

For a very long time used the following desktop entry to force Firefox to
render using a light GTK theme even when using a dark theme for the desktop.

```
[Desktop Entry]
Version=1.0
Name=Proxy
GenericName=Web Proxy
Comment=Proxy the Web
Exec=env GTK_THEME=adwaita:light firefox -P proxy %u
Icon=firefox
Terminal=false
Type=Application
MimeType=text/html;text/xml;application/xhtml+xml;application/vnd.mozilla.xul+xml;text/mml;x-scheme-handler/http;x-scheme-handler/https;
StartupNotify=true
Categories=Network;WebBrowser;
Keywords=web;browser;internet;
```

The downside to this is that the outer Firefox window will render using the
light theme as well. It turns out that there is another config option that
allows you to override the internal page content theme, while allowing Firefox
to use the dark theme to render the window.

```
widget.content.gtk-theme-override = Adwaita:light
```

You have to manually create the configuration key in `about:config`, after that
restart Firefox and you should have normal looking page content with a dark GTK
theme.
