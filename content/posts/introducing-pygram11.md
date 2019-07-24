+++
title = "Introducing pygram11"
date = 2019-03-04
tags = ["python", "numpy", "cpp", "hep"]
draft = false
+++

I'm very happy to release my first real open source software
project: [pygram11](https://github.com/douglasdavis/pygram11). I've been writing software for a while now, but
it's mostly been within the confines of a
physics-experiment-specific use case. In that time I've used a lot
of other developers' software, so it feels quite nice to
potentially help contribute to the scientific computing community
in the same way.

This python library aims to make generating a lot of histograms a
quick task (targeting samples of size \\(O(10^6)\\) and larger), while
supporting weighted statistical uncertainties on the bin counts. To
do this I've implemented the ability to calculate histograms (both
fixed and variable bin width in one and two dimensions) which are
(optionally) accelerated with [OpenMP](https://www.openmp.org/). To do it in Python, I've used
[pybind11](https://github.com/pybind/pybind11). Pygram11 can essentially be a drop-in replacement for
`numpy.histogram` and `numpy.histogram2d`, while reaching speeds
20x faster (for a 1D histogram of an array of length 10,000) to
almost 100x faster than NumPy (for a 2D histogram of 100 million
\\((x\_i, y\_i)\\) pairs). The APIs are quite similar (with slightly
different return styles). On top of that, the
sum-of-weights-squared calculation is a "first class citizen" in
pygram11 (see my [NumPy Histogram tricks for HEP](https://ddavis.io/posts/2018-02-08-numpy-histograms/) post).

So, please go checkout the [documentation](https://pygram11.readthedocs.io/) and [GitHub repository](https://github.com/douglasdavis/pygram11)!
Open issues, PRs, email me, tweet me, or write something even
better.

To try it out (with OpenMP support), all you need is

```nil
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

h_1d = histogram(x, bins=20, range=(-4, 4), omp=True)
h_2d = histogram2d(x, y, bins=[20, 40], range=[[-4, 4], [-3, 3]], omp=True)

h_1d, sumw2_1d = histogram(x, bins=20, range=(-4, 4), weights=w, omp=True)
h_2d, sumw2_2d = histogram2d(x, y, bins=[20, 40], range=[[-4, 4], [-3, 3]], weights=w, omp=True)
```

Notice the sum-of-weights squared is returned if the `weights`
argument is provided with an array of sample weights. Checkout
some [benchmarks](https://pygram11.readthedocs.io/en/stable/purpose.html#some-benchmarks) to see how the `omp` argument can speed up the
calculations.

And some variable bin histogramming, uniform logarithmic:

```python
import numpy as np
from pygram11 import histogram

x = np.exp(np.random.uniform(0.1, 10.0, 100000))
bins = np.logspace(0.1, 1.0, 10, endpoint=True)

h = histogram(x, bins=bins, omp=True)
```


## Future plans {#future-plans}

I hope to eventually spend some time potentially optimizing the
OpenMP usage. I think there is some room for improvement there. I
would also like to add some optional visualization utilities,
we'll see.
