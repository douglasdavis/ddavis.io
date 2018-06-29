---
layout: post
title: "Yet Another Emacs C++ IDE"
date: 2018-06-29
---

I've seen a lot of posts on the
[Emacs](https://old.reddit.com/r/emacs) and
[C++](https://old.reddit.com/cpp) subreddits over the last few months
related to Emacs as a C/C++ IDE. If you give the topic a quick
googling you'll see a lot of posts and tutorials which walk through
using [cquery](https://github.com/cquery-project/cquery),
[lsp-mode](https://github.com/emacs-lsp/lsp-mode),
[rtags](https://github.com/Andersbakken/rtags),
[ggtags](https://github.com/leoliu/ggtags),
[irony](https://github.com/Sarcasm/irony-mode),
[company](http://company-mode.github.io/),
[ycmd](https://github.com/abingham/emacs-ycmd), etc. (obviously there
are a number of options out there and multiple blog posts and
tutorials for each). I've personally tried using cquery and rtags
(both
[libclang](https://github.com/llvm-mirror/clang/tree/master/tools/libclang)
based) in combination with company-mode. Playing with those packages
produced a hacked up Emacs init file and I didn't really know what I
was doing at the time. I was never really satisfied with the black box
I created for myself -- so I decided to clean it up and start over
after some research.

I've recently landed on a new setup using a combination of lsp-mode,
company, and [lsp-clangd](https://github.com/emacs-lsp/lsp-clangd). As
is clear from the package name, this method takes advantage of (the
rapidly in development) LLVM tool
[clangd](https://github.com/llvm-mirror/clang-tools-extra/tree/master/clangd).

I use this setup in combination with `compile_commands.json` files
which are [produced by
CMake](https://cmake.org/cmake/help/latest/variable/CMAKE_EXPORT_COMPILE_COMMANDS.html). This
file must be kept at the project root.

I'm still by no means an expert, but it was (again) a good learning
experience and I no longer have a black box from copying and pasting
other configs. My Emacs `init.el` now has a few simple blocks:


Ensure that `company-lsp` is installed and be sure to turn on
company-mode globally:

```lisp
(use-package company-lsp
  :ensure t
  :config
  (require 'company-lsp)
  (push 'company-lsp company-backends)
  (add-hook 'after-init-hook 'global-company-mode)
  )
```

Ensure that `lsp-mode` and `lsp-ui` are installed and required:

```lisp
(use-package lsp-mode
  :ensure t
  :config
  (require 'lsp-mode)
  )

(use-package lsp-ui
  :ensure t
  :config
  (require 'lsp-ui)
  )
```

Unfortunately `lsp-clangd` isn't in melpa yet, so I cloned it to my
home directory and make sure to point to it. Then add a hook to C++
mode to enable it:

```lisp
(use-package lsp-clangd
  :load-path
  "~/.emacs.d/lsp-clangd"
  :init
  (add-hook 'c++-mode-hook #'lsp-clangd-c++-enable)
  )
```

Finally, I made a small change to `lsp-clangd` to define my `clangd`
executable's full path (as of today, by default, the package requires
the executable to be in your `$PATH`, this could change as the very
new project matures).
