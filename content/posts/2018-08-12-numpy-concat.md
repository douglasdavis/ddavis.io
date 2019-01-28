---
layout: post
title: "Repetitive NumPy Concatenations"
date: 2018-08-12
tags: ["python", "numpy"]
---

I recently had to construct a couple of numpy arrays from a handful of
files. I quickly did something like this:

```python
files = list_of_files()
arr = np.array([], dtype=np.float32)
for f in files:
    iarr = get_arr_from_file(f)
    arr = np.concatenate([arr, iarr])
```

This was taking a lot longer than I thought it should. There's a very
simple reason: NumPy arrays have to be contiguous in memory so I was
copying my `arr` variable `len(files)` times to construct a final
`arr` (and every iteration of the loop `arr` was getting larger). This
was of course unnecessary.

A better way to do it:

```python
files = list_of_files()
arrs = []
for f in files:
    iarr = get_arr_from_file(f)
    arrs.append(iarr)
arr = np.concatenate(arrs)
```

So when it comes to repetitive NumPy concatenations... avoid it.

A quick test in IPython:

```python
import numpy as np


def bad():
    arr = np.array([], dtype=np.float32)
    for i in range(100):
        iarr = np.random.randn(100000)
        arr = np.concatenate([arr, iarr])
    return arr


def good():
    arrs = []
    for i in range(100):
        iarr = np.random.randn(100000)
        arrs.append(iarr)
    arr = np.concatenate(arrs)
    return arr


%timeit bad()
%timeit good()
```

The output:

```none
1.73 s ± 2 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
247 ms ± 2.52 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
```

Quite the difference!
