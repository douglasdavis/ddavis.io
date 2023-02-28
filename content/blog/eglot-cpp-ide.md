+++
title = "Eglot based Emacs C++ IDE with clangd"
date = 2019-01-07
tags = ["emacs", "cpp"]
draft = false
+++

I have an [old post](@/blog/clangd-emacs-ide.md) documenting my first
attempt at turning Emacs into a C++ IDE with `clangd`. That post
describes using two packages: `lsp-mode` and `lsp-clangd`. Those
packages have evolved and now `clangd` usage is built into `lsp-mode`,
so the post is a bit outdated. I've also started to use
[Eglot](https://github.com/joaotavora/eglot) (see previous post for my
Eglot Python IDE). So, let's put together an updated setup:


## Requirements {#requirements}

First, one needs to have `clangd` installed. These days, the 8.0
release of LLVM is a few months away, but `clangd` (part of the
`clang-tools-extra` LLVM project) is in rapid development and the
`master` of the repository should be used. The [installation
instructions](https://llvm.org/docs/GettingStarted.html#for-developers-to-work-with-a-git-monorepo)
from the LLVM documentation are easy to follow.

My C++ development happens on multiple machines. In my Emacs
configuration I keep a simple variable around to point to wherever
`clangd` is installed on various machines.

```lisp
(defvar ddavis-clangd-exe (executable-find "clangd")
  "clangd executable path")
```

By default I'm letting Emacs find it, but I have things like this
sprinkled around my configuration (pointing to a specific LLVM
installation not in my `PATH`):

```lisp
(when (string= (system-name) "pion")
  (setq ddavis-clangd-exe "~/software/llvm/releases/master/bin/clangd"))
```


## Eglot setup {#eglot-setup}

Eglot uses `project.el`, but I use
[Projectile](https://github.com/bbatsov/projectile), so I start by
defining a function that will tell `project.el` to find a project via
Projectile, [thanks @wyuenho on
GitHub](https://github.com/joaotavora/eglot/issues/129#issuecomment-444130367):

```lisp
(defun dd/projectile-proj-find-function (dir)
  (let ((root (projectile-project-root dir)))
    (and root (cons 'transient root))))
```

Now I have a function I call when I'm ready to start digging into a
C++ project which has an associated
[`compile_commands.json`](https://clang.llvm.org/docs/JSONCompilationDatabase.html):

```lisp
(use-package eglot
  :ensure t)

(defun dd/cpp-eglot-enable ()
  "enable variables and hooks for eglot cpp IDE"
  (interactive)
  (setq company-backends
        (cons 'company-capf
              (remove 'company-capf company-backends)))
  (with-eval-after-load 'project
    (add-to-list 'project-find-functions
                 'dd/projectile-proj-find-function))
  (add-to-list 'eglot-server-programs
               `((c++-mode) ,ddavis-clangd-exe))
  (add-hook 'c++-mode-hook 'eglot-ensure))
```

- Ensure that Eglot is installed via `use-package`.
- Ensure that the `completion-at-point` backend is used by `company`
  (bring it to the front of the `company-backends` list).
- Ensure that `project.el` uses Projectile to find my project
  definition (this is because I usually have C++ projects using git
  submodules).
- Add my `clangd` executable to the `eglot-server-programs` list.
- Add the hook to automatically start Eglot.

If I don't want the hook anymore, I use this very simple function:

```lisp
(defun dd/cpp-eglot-disable ()
  "disable hook for eglot"
  (interactive)
  (remove-hook 'c++-mode-hook 'eglot-ensure))
```
