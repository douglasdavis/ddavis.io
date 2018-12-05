+++
title = "New Toy: ox-hugo"
author = ["Douglas Davis"]
date = 2018-12-04
draft = false
+++

I've recently fallen into a very deep Emacs-filled rabbit hole. It
started with the goal of cleaning up my Emacs `init.el` file, but
expanded to learning more Emacs Lisp and trying to get more out of
Org-mode. Now I'm typing this post in Org-mode with a new toy:
[`ox-hugo`](https://ox-hugo.scripter.co/). This Emacs package makes it easy to create blog posts
from a single Org-file by seamlessly exporting second level
headlines to Hugo's Markdown syntax.

Setting up `ox-hugo` was incredibly easy. With MELPA already
configured, the only required addition to my init file was this:

```emacs-lisp
(use-package ox-hugo
  :ensure t
  :after ox)
```

Now, in the buffer I'm currently editing, I use the key-binding
`C-c C-e H H` to export a ready-to-go markdown file for Hugo to
parse.

Luckily before I fell down the `ox-hugo` rabbit hole my Emacs
configuration was already cleaned up to my liking, hopefully it
stays that way for a while! It can be found [right here](https://github.com/drdavis/dotfiles/blob/master/emacs/emacs-init.org) (you'll see
it's also written in Org-mode, with the Emacs Lisp blocks loaded
via `org-babel-load-file`).
