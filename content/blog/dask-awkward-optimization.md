+++
title = "Speeding up dask-awkward... with itself"
date = 2023-03-02
tags = ["python", "dask"]
draft = false
+++

We released the first non-pre-release version of
[dask-awkward](https://github.com/dask-contrib/dask-awkward) a couple
of months ago, but the project is far from done! Something that has
taken a lot of our focus over the last few weeks has been improving
what we call the "necessary columns" optimization. The goal of the
optimization is to avoid wasting compute and memory on unnecessary IO.
This post will describe how the optimization works. I expect the
reader to have some basic familiarity with Dask.

### Projecting columns

Let's consider a simple example: reading a Parquet dataset from disk
into a `pandas.DataFrame` or `dask.dataframe.DataFrame`. The
[Parquet](https://parquet.apache.org/) format defines columns, and we
can decide which columns to read from the dataset when instantiating a
dataframe. If we have a dataset with three columns: "a", "b",
and "c", calling

```python
>>> import pandas as pd
>>> df = pd.read_parquet("/path/to/data")
```

will create a DataFrame with all three columns. We can read only
"b" and "c" with:

```python
>>> df = pd.read_parquet("/path/to/data", columns=["b", "c"])
```

This is beneficial if we know exactly which columns we need to compute
our desired result. In the case above, perhaps we just need the ratio

```python
>>> result = df["c"] / df["b"]
```

Reading the entire dataset from disk (that is, including "a")
wastes compute (more IO) and memory (unnecessary data stored in the
dataframe).

Pandas is an eager library and each function call is going to run some
compute (like reading data from disk). Pandas cannot look into the
future to see which columns are actually necessary in your program
such that it only reads those columns.

In [dask.dataframe](https://docs.dask.org/en/stable/dataframe.html),
column projection occurs when the task graph detects that some kind of
column selection (via a Python `getitem` call on a dataframe) has
occurred. When working with Dask, your workflow builds up a task
graph, staging future computation. When we call `read_parquet` with
`dask.dataframe`, we are not immediately reading the data from disk,
we are staging that IO step.


```python
>>> df = dd.read_parquet("/path/to/data")
```

At this point, if we compute this `dask.dataframe` (with
`df.compute()`), all columns will be read. We can follow the same
procedure above and manually define which columns we want using the
`columns=` argument. If you have a complex dataset with tens or
hundreds of columns and you don't necessarily know what you need yet,
it's hard to define `columns=`. Dask can help. If we write:

```python
>>> import dask.dataframe as dd
>>> df = dd.read_parquet("/path/to/data")
>>> result = df["c"] / df["b"]
```

We get a Dask task graph with four steps:

1. Read the Parquet data at "/path/to/data" (no columns specified).
2. Get column "c" Series.
3. Get column "b" Series.
4. Divide column "c" by "b".

**None of these steps happen until we call `result.compute()`**

Because of the lazy computation, when we call `.compute()`, Dask can
parse the task graph _before_ computing step 1 to see that we will use
Python `geitem` calls to grab _only_ columns "c" and "b". Dask will
then update step 1 in the graph to **automatically** pass in a
`columns=` argument to Pandas when it calls the `pd.read_parquet`
function at the compute node. The task graph is rewritten to have step
1 be:

1. Read the Parquet data at "/path/to/data" (now with the argument
   `columns=["b", "c"]`)

with the remaining steps unchanged. This was simple example of column
projection in `dask.dataframe`. It is one optimization that is
executed when Dask performs a compute. Please [read more about Dask
optimizations in
general](https://docs.dask.org/en/stable/optimize.html) if you are
interested.
