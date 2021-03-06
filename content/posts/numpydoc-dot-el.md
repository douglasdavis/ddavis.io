+++
title = "NumPy style docstrings with numpydoc.el"
date = 2021-02-28
tags = ["python", "numpy", "emacs"]
draft = false
+++

The majority of Python code I write uses the scientific/PyData
ecosystem. This leads to reading and writing a lot of [NumPy style
docstrings](https://numpydoc.readthedocs.io/en/latest/format.html). If you've read some of my other posts you've undoubtedly
noticed my editor of choice is GNU Emacs. Docstring-generating Emacs
Lisp packages for Python do exist: [sphinx-doc.el](https://github.com/naiquevin/sphinx-doc.el) and [docstr](https://github.com/jcs-elpa/docstr). The
former does not support the NumPy docstring style, while the latter
isn't quite as plug-and-play as I wanted (docstr supports _many_
programming languages and it's very programmable).

I wrote [numpydoc.el](https://github.com/douglasdavis/numpydoc.el) to support the features I was looking for. Simply
call `numpydoc-generate` from the body of the function of interest to
create a docstring. The function signature and body are parsed to
determine argument names, types, default values, return type and what
exceptions (if any) can be raised. That information is used to
generate the `Parameters` block and the `Returns` block for the
docstring. If exceptions are found the `Raises` block is also added.
The default behavior prompts the user to enter a short and long
function description along with descriptions for the individual
components of the function. You can also configure the package to use
[yasnippet](https://github.com/joaotavora/yasnippet). The git repository README includes a gif with some example
usage. Who doesn't want to turn this:

```python
def func(number: int = 5, label: Optional[str] = None) -> str:
    if number > 42:
        raise ValueError("Illegal number")
    if label is not None:
        return label * number
    return "None" * number
```

into this:

```python
def func(number: int = 5, label: Optional[str] = None) -> str:
    """FIXME: Short description.

    FIXME: Long description.

    Parameters
    ----------
    number : int
        FIXME: Add docs.
    label : Optional[str]
        FIXME: Add docs.

    Returns
    -------
    str
        FIXME: Add docs.

    Raises
    ------
    ValueError
        FIXME: Add docs.

    Examples
    --------
    FIXME: Add docs.

    """
    if number > 42:
        raise ValueError("Illegal number")
    if label is not None:
        return label * number
    return "None" * number
```

with one `M-x` execution? The package is [available on MELPA](https://melpa.org/#/numpydoc). Just add
to your (`use-package` leveraging) `init.el`:

```emacs-lisp
(use-package numpydoc
  :ensure t
  :after python)
```

Perhaps you can bind it to `C-c C-n` (it's a vacant binding, unused by
`python.el` as of writing this post):

```emacs-lisp
(use-package numpydoc
  :ensure t
  :after python
  :bind (:map python-mode-map
              ("C-c C-n" . numpydoc-generate)))
```
