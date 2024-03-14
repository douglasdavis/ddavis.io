+++
title = "Eglot based Emacs Python IDE"
date = 2018-12-05
draft = false
[taxonomies]
tags = ["emacs", "python"]
+++

**This post is now a bit outdated** [an updated post is here](@/blog/emacs-python-lsp.md).

In my Emacs rabbit hole I mentioned in my previous post, I decided to
work on improving my Python development workflow. I recently found the
[Eglot](https://github.com/joaotavora/eglot) package for running a
[LSP](https://microsoft.github.io/language-server-protocol/) in Emacs.

The most vanilla setup for Eglot is just `M-x eglot` in a buffer
editing a python file. This works wonderfully if the executable for
the [Python Language
Server](https://github.com/palantir/python-language-server) (`pyls`)
is found. This works because Eglot defines a list of server programs
by default. See this list with `M-: eglot-server-programs`


## Project Editing {#project-editing}

I have a few python virtual/Anaconda environments I like to work with.
This is what `.dir-locals.el` is for:

```lisp
((python-mode . ((eglot-server-programs    . ((python-mode "/path/to/env/bin/pyls")))
                 (python-shell-interpreter . "/path/to/env/bin/python")
                 (company-backends         . (company-capf)))))
```

where `/path/to/env` is the path to a virtual environment or Anaconda
environment (that of course has `python-language-server` installed). I
also define the path to my Python executable for Emacs' builtin
`python.el`. By default, `company-backends` includes `company-capf`
for `completion-at-point`, but I want to make sure that's what is used
because Eglot provides `completion-at-point`. Eglot also has `pyls` as
a `python-mode` entry by default, but not to the virtual environment I
want to use; this is why I manually define the list of server
programs.

When I open a buffer in the project I want to work in, I just call
`M-x eglot` and I'm up and running.


## Non-project Editing {#non-project-editing}

If I'm not editing in a project that has an associated virtual
environment, I rely on some "sensible defaults" in my Emacs init file:

```lisp
(defvar ddavis-default-pyls "~/software/Python/anaconda3/bin/pyls"
  "define a default pyls to be used")
```

This way I have a default `pyls` executable from my `base` Anaconda
environment (which is potentially different on different machines). I
then have a couple of functions to handle default Eglot python
environments, where I:

-   Make `use-package` install Eglot if necessary.
-   Make sure `company-capf` is at the front of `company-backends`.
-   Make sure I add an Eglot server program entry pointing to my
    `base` Anaconda `pyls` to the front of the `eglot-server-programs`
    list.
-   Add the desired hook.

<!--listend-->

```lisp
(use-package eglot
  :ensure t)

(defun dd/python-eglot-enable ()
  "set variables and hook for eglot python IDE"
  (interactive)
  (setq company-backends
        (cons 'company-capf
              (remove 'company-capf company-backends)))
  (add-to-list 'eglot-server-programs
               `(python-mode ,ddavis-default-pyls))
  (add-hook 'python-mode-hook 'eglot-ensure))

(defun dd/python-eglot-disable ()
  "remove hook for eglot python"
  (interactive)
  (remove-hook 'python-mode-hook 'eglot-ensure))
```

I just bring `company-capf` to the front of the `company-backends`
list, and add my desired Anaconda based `pyls` to front of the
`eglot-server-programs` list.
