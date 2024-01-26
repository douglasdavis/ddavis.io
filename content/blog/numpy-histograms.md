+++
title = "NumPy Histogram tricks for HEP"
date = 2018-02-08
tags = ["hep", "numpy", "python"]
draft = false
[extra]
katex = true
+++

**Update August 2019**: About a year after writing this blog post I
created a Python package to handle all of my pythonic histogramming
needs. It's called [pygram11](https://github.com/douglasdavis/pygram11). This post is definitely still useful
for learning more details about NumPy histogramming.


## Our starting point {#our-starting-point}

Histogramming some data is simple using [numpy.histogram](https://docs.scipy.org/doc/numpy/reference/generated/numpy.histogram.html).

```python
>>> import numpy as np
>>> x = np.random.randn(10000)           ## create a dataset
>>> w = np.random.normal(1, 0.2, 10000)  ## create some phony weights
>>> b = np.linspace(-5, 5, 11)           ## bin edges (10 bins from -5 to 5)
>>> n, bins = np.histogram(x, bins=b, weights=w)
```

This gives me two arrays

-   one for the bin heights (`n`)
-   one for the bin edges (`bins`).

Quick and simple -- but what if I want to include underflow and
overflow in the first and last bins, respectively? What if I want
to compute the error on each bin height given a weighted dataset?
These quantities are important for high energy physics, where
nearly all of our analysis is done using histograms.


## Underflow and overflow {#underflow-and-overflow}

Where the elements of the data contribute to the bin height is of
course determined by the bin edges. We can make the left and right
edges infinite to be sure to include _all_ of our data[^1]. Then we
just add the `[0]` bin contents to the `[1]` bin contents, and add the
`[-1]` bin contents to the `[-2]` bin contents. Finally, we polish it
off by chopping off the out-of-bounds elements:

```python
>>> import numpy as np
>>> raw_bins = np.linspace(-5, 5, 11)
>>> use_bins = [np.array([-np.inf]), raw_bins, np.array([np.inf])]
>>> use_bins = np.concatenate(use_bins)
>>> x = np.random.normal(0, 2, 1000) ## phony dataset
>>> n, bins = np.histogram(x, bins=use_bins)
>>> n[1]  += n[0]   ## add underflow to first bin
>>> n[-2] += n[-1]  ## add overflow to last bin
>>> n = n[1:-1]     ## chop off the under/overflow
>>> bins = raw_bins ## use our original binning (without infinities)
```

And that's it, now _all_ of the data is histogrammed -- including
under and overflow.


## Error on bin height using weights {#error-on-bin-height-using-weights}

The standard error on a bin height is simply the square-root of
the bin height, \\(\sqrt{N}\\)[^2]. If a bin is constructed from
weighted data, we require the square-root of the sum of the
weights squared, \\(\sqrt{\sum\_i w\_i^2}\\).

The `numpy.histogram` function doesn't provide any information
about which weights belong to which bin, but we have another
useful NumPy function which can generate an array of indices based
on where data falls in a particular set of bins, [numpy.digitize](https://docs.scipy.org/doc/numpy/reference/generated/numpy.digitize.html).

First, we get an array representing which bin each data point
would fall into. We can then use the conditional function
[numpy.where](https://docs.scipy.org/doc/numpy/reference/generated/numpy.where.html) in a loop over all bins to grab only the weights in
that bin, and sum their squares.

```python
>>> import numpy as np
>>> x = np.random.normal(0, 2.0, 1000)         ## a dataset
>>> b = np.linspace(-2, 2, 21)                 ## 20 bins
>>> w = np.random.normal(1, 0.2, 1000)         ## some weights
>>> sum_w2 = np.zeros([20], dtype=np.float32)  ## start with empty errors
>>> digits = np.digitize(x, b)                 ## bin index array for each data element
>>> for i in range(nbins):
>>>     weights_in_current_bin = w[np.where(digits == i)[0]]
>>>     sum_w2[i] = np.sum(np.power(weights_in_current_bin, 2))
>>> n, bins = np.histogram(x, bins=b, weights=w)
>>> err = np.sqrt(sum_w2)
```

Now two arrays exist: `n` contains the heights in each bin, and
`err` contains the standard error on the bin heights.


## Appendix, a function to combine the two methods: {#appendix-a-function-to-combine-the-two-methods}

```python
def extended_hist(
    x: np.ndarray
    nbins: int
    range: Tuple[float, float],
    underflow: bool = True,
    overflow: bool = True,
    weights: Optional[np.ndarray] = None,
) -> Tuple[np.ndarray, np.ndarray, np.ndarray, np.ndarray]:
    """Histogram weighted data with potential under/overflow.

    Parameters
    ----------
    x : array_like
        Data to histogram.
    nbins : int
        Total number of bins.
    range : (float, float)
        Definition of binning max and min.
    underflow : bool
        Include undeflow data in the first bin.
    overflow : bool
        Include overflow data in the last bin.
    weights : array_like, optional
        Weights associated with each element of ``x``.

    Returns
    -------
    numpy.ndarray
        Total bin values.
    numpy.ndarray
        Poisson uncertainty on each bin count.
    numpy.ndarray
        Bin centers.
    numpy.ndarray
        Bin edges.

    """
    if weights is not None:
        if weights.shape != x.shape:
            raise ValueError(
                "Unequal shapes x: {}; weights: {}".format(
                    x.shape, weights.shape
                )
            )
    xmin, xmax = range
    edges = np.linspace(xmin, xmax, nbins + 1)
    neginf = np.array([-np.inf], dtype=np.float32)
    posinf = np.array([np.inf], dtype=np.float32)
    bins = np.concatenate([neginf, edges, posinf])
    if weights is None:
        hist, bin_edges = np.histogram(x, bins=bins)
    else:
        hist, bin_edges = np.histogram(x, bins=bins, weights=weights)

    n = hist[1:-1]
    if underflow:
        n[0] += hist[0]
    if overflow:
        n[-1] += hist[-1]

    if weights is None:
        u = np.sqrt(n)
    else:
        bin_sumw2 = np.zeros(nbins + 2, dtype=np.float32)
        digits = np.digitize(x, edges)
        for i in range(nbins + 2):
            bin_sumw2[i] = np.sum(
                np.power(weights[np.where(digits == i)[0]], 2)
            )
        u = bin_sumw2[1:-1]
        if underflow:
            u[0] += bin_sumw2[0]
        if overflow:
            u[-1] += bin_sumw2[-1]
        u = np.sqrt(u)

    centers = np.delete(edges, [0]) - (np.ediff1d(edges) / 2.0)
    return n, u, centers, edges
```

**Update August 2019**: With `pygram11`, we can just import the
`histogram` function and call a one-liner for the values and the
error:

```python
>>> from pygram11 import histogram
>>> data, weights = get_some_weighted_data()
>>> h, err = histogram(data, bins=10, range=(xmin, xmax), weights=weights, flow=True)
```

---

[^1]: In the implementation of `numpy.histogram`, elements of the
    input array that live outside the bounds of the binning are
    ignored.

[^2]: Bin height is related to counting, therefore the data in a bin
    is
    [Poissonian](https://en.wikipedia.org/wiki/Poisson_distribution).
    The variance of a Poisson distribution is \\(N\\).
