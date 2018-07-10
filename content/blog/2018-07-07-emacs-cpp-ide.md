---
layout: post
title: "Clangd based Emacs C++ IDE"
date: 2018-07-07
---

I've seen a lot of posts on the
[Emacs](https://old.reddit.com/r/emacs) and
[C++](https://old.reddit.com/cpp) subreddits over the last few months
related to Emacs as a C/C++ IDE. If you give the topic a quick
googling you'll see a lot of tutorials that walk through using
[cquery](https://github.com/cquery-project/cquery),
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
was doing at the time. I was never comfortable with the black box I
created for myself -- so I decided to clean it up and start over after
some research.

I've recently landed on a new setup using a combination of lsp-mode,
company, and [lsp-clangd](https://github.com/emacs-lsp/lsp-clangd). As
is clear from the package name and post title, this method takes
advantage of the LLVM/Clang tool
[clangd](https://github.com/llvm-mirror/clang-tools-extra/tree/master/clangd)
(which is very much in development).

Here's a quick rundown of the new configuration:

Ensure that `company-lsp` is installed and enable company-mode (I
choose a global configuration):

```lisp
(use-package company-lsp
  :ensure t
  :config
  (require 'company-lsp)
  (push 'company-lsp company-backends)
  (add-hook 'after-init-hook 'global-company-mode))
```

Ensure that `lsp-mode` and `lsp-ui` are installed and required:

```lisp
(use-package lsp-mode
  :ensure t
  :config
  (require 'lsp-mode))

(use-package lsp-ui
  :ensure t
  :config
  (require 'lsp-ui))
```

Unfortunately `lsp-clangd` isn't in melpa yet, so I cloned it to my
`.emacs.d` directory and make sure to point to it (while writing this
post there is an [open GitHub
PR](https://github.com/melpa/melpa/pull/5593) to add lsp-clangd to
melpa). Be sure to set the proper clangd executable path and add a
hook to C++ mode to enable it:

```lisp
(use-package lsp-clangd
  :load-path
  "~/.emacs.d/lsp-clangd"
  :init
  ;; for macOS
  (when (equal system-type 'darwin)
    (setq lsp-clangd-executable "/usr/local/opt/llvm/bin/clangd"))
  ;; for Fedora box
  (when (string= (system-name) "proton")
    (setq lsp-clangd-executable "/home/ddavis/Software/llvm/head/bin/clangd"))

  (add-hook 'c++-mode-hook #'lsp-clangd-c++-enable))
```

Like I said, Clangd is under heavy development, so expect some
imperfections. For example, using the version shipped with the LLVM
6.0.0 release wasn't working with header files. I went ahead and built
a bleeding edge installation (using `brew install --HEAD llvm` on macOS
and building from the trunk of their svn repositories on a Fedora
machine; read how to do that
[here](http://clang.llvm.org/get_started.html)) and that fixed the
problem.

I use this setup in combination with `compile_commands.json` files
which are [produced by
CMake](https://cmake.org/cmake/help/latest/variable/CMAKE_EXPORT_COMPILE_COMMANDS.html). This
file must be kept at the project root (using
[Projectile](https://github.com/bbatsov/projectile) with a
`.projectile` file at the project root helps when using git
repositories with submodules; lsp-mode appears to handle that nicely).

I'm still by no means an expert, but it was a good learning experience
and I no longer have a black box from copying and pasting from other's
Emacs init files. I have code completion and inter/intra-project file
and definition jumping -- the two big features I like to add to my C++
development setup in Emacs.
