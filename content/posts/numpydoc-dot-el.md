+++
title = "NumPy style docstrings with numpydoc.el"
date = 2021-02-27
tags = ["python", "numpy", "emacs"]
draft = false
+++

The majority of Python code I write uses the scientific/PyData
ecosystem. This leads to reading and writing a lot of [NumPy style
docstrings](https://numpydoc.readthedocs.io/en/latest/format.html). If you've read some of my other posts you've undoubtedly
noticed my editor of choice is GNU Emacs. A package to generate a
skeleton for Sphinx style docstrings does exist: [sphinx-doc.el](https://github.com/naiquevin/sphinx-doc.el); but it
lacks support for the numpydoc style and does not support a couple of
other features I'm looking for: customizations, type hint support, and
automatic formatting of user-entered text for the components of the
docstring. There's also the [docstr](https://github.com/jcs-elpa/docstr) package, which caters to any
programming language. I wanted something a bit more plug-and-play.

I wrote [numpydoc.el](https://github.com/douglasdavis/numpydoc.el) to support the features I was looking for. One
simply calls `numpydoc-generate` from the body of the function of
interest to create a docstring. The function signature and body are
parsed to determine argument names, types, default values, return type
and what exceptions (if any) can be raised. That information is used
to generate the `Parameters` block and the `Returns` block for the
docstring. If exceptions are found the `Raises` block is also added.
The default behavior prompts the user to enter a short and long
function description along with descriptions for the individual
components of the function. The git repository README includes a gif
with some example usage.

The package is [available on MELPA](https://melpa.org/#/numpydoc.el). Just add to your (`use-package`
leveraging) `init.el`:

```emacs-lisp
(use-package numpydoc
  :ensure t
  :after python)
```
