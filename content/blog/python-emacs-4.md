+++
title = "Python & Emacs, 2025"
date = 2025-08-27
draft = false
[taxonomies]
tags = ["python", "emacs"]
+++

Here we go with installment four of the unofficial series
([first](@/blog/eglot-python-ide.md),
[second](@/blog/emacs-python-lsp.md),
[third](@/blog/python-emacs-3.md)). I've decided I'll start putting
the year in the title.

Back in 2022 [`python-base-mode` was
added](https://github.com/emacs-mirror/emacs/commit/00df4566af9dff0a27fd6da566ef1e53268a6d47)
to Emacs such that (a) `python-mode` and `python-ts-mode` can inherit
a lot of shared functionality, and (b) users don't have to account for
customizing both modes (when their configuration needs to be used in
both tree-sitter and non-tree-sitter enabled Emacs builds). It's a
good idea to use the generic major mode when possible.

These days I'm on the [Astral](https://astral.sh/) train, which really
means `ruff` and `uv` (and `ty`, sort of, since they're not done yet).

Unfortunately `ruff` isn't able to run both `format` and the import
sorting mechanism of the linter (`ruff check`) in the same command. So
I have a couple of reformatters defined to handle both steps:

```
(use-package reformatter
  :ensure t
  :config
  (reformatter-define dd/ruff-format
    :program "uvx"
    :args `("ruff" "format" "--stdin-filename" ,buffer-file-name "-"))
  (reformatter-define dd/ruff-sort
    :program "uvx"
    :args `("ruff" "check" "--select" "I" "--fix" "--stdin-filename" ,buffer-file-name "-")))
```

The default virtual environment path for a project managed with `uv`
is just `.venv`. Combining that behavior with my reformatters, I get a
simple "init" function for all of my Python buffers. The function
automatically finds `.venv` in the project and calls `pyvenv-activate`
(from the [pyvenv](https://github.com/jorgenschaefer/pyvenv) package);
it also enables the reformatters:

```
(defun dd/python-init ()
  (let* ((project (project-current))
         (project-root (when project (project-root project)))
         (venv-path (when project-root
                      (expand-file-name ".venv" project-root))))
    (when (and venv-path (file-directory-p venv-path))
      (make-local-variable 'pyvenv-virtual-env)
      (pyvenv-activate venv-path))
    (dd/ruff-format-on-save-mode +1)
    (dd/ruff-sort-on-save-mode +1)))

(add-hook 'python-base-mode-hook #'dd/python-init)
```

The reason I need to "activate" the virtual environment in Emacs is
because I typically install whatever language servers I want to play
with in the virtual environment of a project. Then, invoking <kbd>M-x
eglot</kbd> will give us choices based on whatever `eglot` finds
(through its `executable-find` usage in `eglot-alternatives` calls
within `eglot-server-programs`). My Python incantation:

```
(add-to-list 'eglot-server-programs
             `(python-base-mode
               . ,(eglot-alternatives '(("basedpyright-langserver" "--stdio")
                                        ("ty" "server")
                                        ("pyright-langserver" "--stdio")))))
```

You'll notice that I cycle through using
[basedpyright](https://github.com/DetachHead/basedpyright),
[ty](https://github.com/astral-sh/ty), and
[pyright](https://github.com/microsoft/pyright).

I'm hopeful that sometime in the near future my `eglot-alternatives`
list can be redefined to

```
(add-to-list 'eglot-server-programs
             `(python-base-mode
               . ,(eglot-alternatives '(("uv" "run" "basedpyright-langserver" "--stdio")
                                        ("uv" "run" "ty" "server")
                                        ("uv" "run" "pyright-langserver" "--stdio")))))
```

such that I can rely on `uv`'s automatic virtual environment
discovery/usage, removing the need to have code that activates a
virtual environment. The current `eglot-alternatives` implementation
in Emacs uses the first entry of the list you provide to pass along to
`completing-read`, and `completing-read` ignores duplicates (and all
entries are duplicates in the list above). I filed
[79290](https://debbugs.gnu.org/cgi/bugreport.cgi?bug=79290); maybe
it'll go somewhere eventually, but it's not a big deal.
