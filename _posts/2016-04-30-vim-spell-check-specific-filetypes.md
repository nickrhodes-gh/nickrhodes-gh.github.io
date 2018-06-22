---
layout: post
title: Enable vim spell check for certain file types
description: >
  How to enable spell checking in Vim for file types specified in vimrc

---

It's not useful to have the Vim spell checker enabled for ever file edited.
Luckily it's straight forward to enable/disable options using autocmd. The
autocmd allows you to hook into certain Vim states as it opens/manipulates
buffers, then run a command if the state matches what is in the autocmd.

The Vim [spell](http://vimdoc.sourceforge.net/htmldoc/spell.html) docs contain
the basic command structure to enable spell check. Docs on autocmd syntax can
be found [here](http://vimdoc.sourceforge.net/htmldoc/autocmd.html).

The format is actually pretty basic.

```
" event: File event to check (BufNewFile/BufRead/FileType)
" pattern: Match pattern (*.html/markdown)
" command: Vim command to be run

au {event} {pattern} {command}
```

To set set spell checking for `markdown` and `gitcommit` file types only, I
think you can only use the `FileType` option due to gitcommits not having file
extensions.

```
au FileType markdown,gitcommit setlocal spell spelllang=en_gb
```

`setlocal` is used so that spell checking is only enabled for the opened
buffer.

Some interesting `{events}` examples taken from the manual `:h
autocmd-events-abc`.

```
BufNewFile			When starting to edit a file that doesn't
				exist.  Can be used to read in a skeleton
				file.

BufRead or BufReadPost		When starting to edit a new buffer, after
				reading the file into the buffer, before
				executing the modelines.  See |BufWinEnter|

BufEnter			After entering a buffer.  Useful for setting
				options for a file type. 

ColorScheme			After loading a color scheme. |:colorscheme|
				The pattern is matched against the
				colorscheme name.

FileType			When the 'filetype' option has been set.  The
				pattern is matched against the filetype.
```

