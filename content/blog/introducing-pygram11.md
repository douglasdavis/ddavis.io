+++
title = "Introducing pygram11"
date = 2019-03-04
tags = ["python", "numpy", "cpp", "hep"]
draft = false
+++

I'm very happy to release my first open source software project:
[pygram11](https://github.com/douglasdavis/pygram11). I've been writing software for a while now, but mostly
targeting physics-experiment-specific use cases. In that time I've
used a lot free and open source software; it feels quite nice to
potentially help contribute to the scientific computing community in
the same way.

This python library aims to make generating many histograms a quick
task (targeting sample sizes above about 10,000 elements), while also
supporting weighted statistical uncertainties on the bin values. Fixed
and variable bin width histograms can be calculated. The backend
implementation is in C++ and accelerated with [OpenMP](https://www.openmp.org/), with [pybind11](https://github.com/pybind/pybind11)
used to generate the Python bindings.

Pygram11 can be a near drop-in replacement for `numpy.histogram` and
`numpy.histogram2d`, while reaching speeds 20x faster (for a 1D
histogram of an array of length 10,000) to almost 100x faster than
NumPy (for a 2D histogram of 100 million \\((x\_i, y\_i)\\) pairs). The APIs
are quite similar (with slightly different return styles). In addition
to the faster calculations, constructing the variance in each bin is a
"first class citizen" in pygram11 (see my [NumPy Histogram tricks for
HEP](@/blog/numpy-histograms.md) post).

So, please go checkout the [documentation](https://pygram11.readthedocs.io/) and [GitHub repository](https://github.com/douglasdavis/pygram11). Open
issues, PRs, email me, tweet at me, or write something better
(checkout some [benchmarks](https://pygram11.readthedocs.io/en/stable/bench.html) in the documentation). To try it out, spin
up a virtual environment or conda environment and install with:

```
pip install pygram11
```

or

```
conda install pygram11 -c conda-forge
```


## In action {#in-action}

Some fixed bin histogramming:

```python
import numpy as np
from pygram11 import histogram, histogram2d

x = np.random.randn(100000)
y = np.random.randn(100000)
w = np.random.uniform(0.8, 1.2, 100000)

h_1d, _ = histogram(x, bins=20, range=(-4, 4))
h_2d, _ = histogram2d(x, y, bins=(20, 40),
                      range=((-4, 4), (-3, 3)))

h_1d, err_1d = histogram(x, bins=20, range=(-4, 4), weights=w)
h_2d, err_2d = histogram2d(x, y, bins=(20, 40),
                           range=((-4, 4), (-3, 3)), weights=w)
```

Notice the error (square-root of the variance) is the second
return object (for the unweighted histogram we just throw it away
with an underscore).

And some variable bin histogramming, uniform logarithmic:

```python
import numpy as np
from pygram11 import histogram

x = np.exp(np.random.uniform(0.1, 10.0, 100000))
bins = np.logspace(0.1, 1.0, 10, endpoint=True)

h, _ = histogram(x, bins=bins)
```
