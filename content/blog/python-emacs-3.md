+++
title = "Python & Emacs, Take 3"
date = 2021-05-26
draft = false
[taxonomies]
tags = ["python", "emacs"]
+++

This is the third installment of what has turned into an unofficial
series of posts on using Emacs for Python development
([first](@/blog/eglot-python-ide.md) and
[second](@/blog/emacs-python-lsp.md) posts). I probably shouldn't
expect it to be the last.

At this point I use [Eglot](https://github.com/jaoatavora/eglot) and
[lsp-mode](https://github.com/emacs-lsp/lsp-mode) pretty
interchangeably. I just like both of them and it's fun to mix it up. I
also continue to use [pyenv](https://github.com/pyenv/pyenv) for
managing python versions and virtual environments and
[pyvenv](https://github.com/jorgenschaefer/pyvenv) for handling them
inside of Emacs.

What has triggered writing a new post is the development of a new
Python language server:
[python-lsp-server](https://github.com/python-lsp/python-lsp-server)
(or `pylsp`). The old Palantir `python-language-server` (`pyls`)
project is no longer maintained. Both Eglot and lsp-mode now support
`pylsp` ([lsp-mode
PR](https://github.com/emacs-lsp/lsp-mode/pull/2846) and [Eglot
commit](https://github.com/joaotavora/eglot/commit/a5b7b7d933b97db9ce5f8b7dcc8c866f7c35b220)).

Beyond `pylsp`, there are a number of other Python language servers
that I've played around with:

- [jedi-language-server](https://github.com/pappasam/jedi-language-server)
- [Pyright](https://github.com/microsoft/pyright)
- [anakin-language-server](https://github.com/muffinmad/anakin-language-server)
- Certainly more...
