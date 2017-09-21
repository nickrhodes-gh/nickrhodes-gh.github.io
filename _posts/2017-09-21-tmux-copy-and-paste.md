---
layout: post
title: Using the mouse to copy and paste in tmux
description: >
  When using mouse mode in tmux it's not possible by default to copy from the
  tmux buffer to the x-clipboard. Tmux does support custom copy commands
  however which make this pretty straight forward.

---

I've started running tmux with `set-option -g mouse on` as it really helps
when your hand is already on the mouse and you just want to quickly scroll up
several lines inside the tmux buffer. It's also useful for a quick copy and
paste into a none terminal window.

The issue is that normal text selection then falls under tmux's control -
basically rendering it usable only within tmux. A workaround is to use
shift-click which passes control back to X, however there is another way.

```
bind-key -T copy-mode-vi MouseDragEnd1Pane send -X copy-pipe-and-cancel 'xclip -i -sel p -f | xclip -i -sel c'
```

I'm guessing this is only supported in some tmux version 2.0+ as it's not
available in 1.8. The tmux 2.5 man pages contain the following mouse events
which are picked up by tmux.

```
WheelUp       WheelDown                  
MouseDown1    MouseUp1      MouseDrag1   MouseDragEnd1
MouseDown2    MouseUp2      MouseDrag2   MouseDragEnd2
MouseDown3    MouseUp3      MouseDrag3   MouseDragEnd3
DoubleClick1  DoubleClick2  DoubleClick3 WheelUp      
TripleClick1  TripleClick2  TripleClick3 WheelDown    
```

These are not the complete event names as they need to be suffixed with a
location, for example `MouseDrag1Window`. Or as above, when we exit a mouse
drag in  a pane it will pipe the selected text to the following command `xclip
-i -sel p -f | clip -i -sel c`. To break this down, there are several X
clipboards available.

```
-selection         
   specify  which  X  selection  to use, options are "primary" to use XA_PRIMARY (default),
   "secondary" for XA_SECONDARY or "clipboard" for XA_CLIPBOARD                            

``` 

If you want to paste using mouse wheel click and `ctrl-alt v` then you need to
pipe the selected text into the `primary` clipboard as well as the
`clipboard`. The `-i` option tells xclip to expect input from stdin and the
`-f` flag tells xclip to write the input text back out to stdout which is how
we are able to daisy chain the xclip commands.
