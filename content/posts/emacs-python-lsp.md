+++
title = "Emacs Python IDE with lsp-mode and py(v)env"
date = 2020-02-18
tags = ["python", "emacs"]
draft = false
+++

I have an [old post]({{< relref "eglot-python-ide" >}}) describing how to spin up an IDE-like Python
development environment in Emacs with [Eglot](https://github.com/joaotavora/eglot) and some
`.dir-locals.el` help. Now a year later, I've converged on what I
think is a better setup.


## pyenv {#pyenv}

My main driver for installing different versions of Python and
spinning up virtual environments is [pyenv](https://github.com/pyenv/pyenv). I use the [automatic](https://github.com/pyenv/pyenv-installer)
installer on all machines where I install pyenv, and I manually
modify my shell's initialization such that I have to execute a
`setupPyenv` function to enable its usage (I also give myself to
ability to activate an environment via a single argument):

```bash
function setupPyenv() {
    export PATH="$HOME/.pyenv/bin:$PATH"
    eval "$(pyenv init -)"
    eval "$(pyenv virtualenv-init -)"
    VENV=$1
    if [ -n "$VENV" ]; then
        pyenv activate $VENV
    fi
}
```


## pyvenv {#pyvenv}

To activate various Python environments in Emacs I turn to
[pyvenv](https://github.com/jorgenschaefer/pyvenv). Since the `pyenv` installer puts itself in your home
directory, we can configure `pyvenv` to find virtual environments
in `~/.pyenv/versions` via the `WORKON_ON` environment variable.
I lean on `use-package` to initialize `pyvenv` and set the
environment variable:

```emacs-lisp
(use-package pyvenv
  :ensure t
  :config
  (setenv "WORKON_HOME" "~/.pyenv/versions"))
```

By setting the `WORKON_HOME` environment variable we can select
which `pyenv` virtual environment we want to use by calling `M-x
    pyvenv-workon`. One can also call `M-x pyvenv-activate` to choose
an environment via manual filesystem navigation.


## lsp-mode {#lsp-mode}

With a `pyvenv` environment activated in Emacs, all we have to do
is call `M-x lsp` (after setting it up of course); [lsp-mode](https://github.com/emacs-lsp/lsp-mode) can be
configured in your `init.el` with something as simple as:

```emacs-lisp
(use-package lsp-mode
  :ensure t
  :commands lsp)
```

See the GitHub project for more details. The virtual environment
you want to use will have to have a language server installed. The
easiest and fastest way to get started (a simple `pip install`) is
to use [pyls](https://github.com/palantir/python-language-server).


## Automated helper {#automated-helper}

Just about all of my Python development happens inside of a
[projectile](https://github.com/bbatsov/projectile) project. I have a simple interactive function that will
automatically activate the environment associated with a project
and spin up lsp-mode.

```emacs-lisp
(defun ddavis/get-pyvenv-name ()
  "grab the name of the active pyvenv (nil if not defined)"
  (when pyvenv-virtual-env
    (car (last (split-string (directory-file-name pyvenv-virtual-env) "/")))))

(defun ddavis/py-auto-lsp ()
  "turn on lsp mode in a Python project by trying to
automatically determine which pyenv virtual environment to
activate based on the project name"
  (interactive)
  (if (and pyvenv-virtual-env
           (file-directory-p pyvenv-virtual-env)
           (string= projectile-project-name (ddavis/get-pyvenv-name)))
      (lsp)
    (pyvenv-workon (projectile-project-name))
    (if (file-directory-p pyvenv-virtual-env)
        (lsp)
      (message (format "%s does not exist, set env manually"
                       pyvenv-virtual-env))
      (call-interactively #'pyvenv-workon)
      (lsp))))
```
