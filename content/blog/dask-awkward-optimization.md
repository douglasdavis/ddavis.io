+++
title = "Using dask-awkward to speed up dask-awkward"
date = 2023-04-10
tags = ["python", "dask"]
draft = false
+++

We released the first non-pre-release version of
[dask-awkward](https://github.com/dask-contrib/dask-awkward) a couple
of months ago, but the project is far from done! Something that has
taken a lot of our focus over the last few weeks has been improving
what we call the "necessary columns" optimization. The goal of the
optimization is to avoid wasting compute and memory on unnecessary
disk reads. This post will describe how the optimization works. I'm
writing this with the expectation that the reader has some basic
familiarity with [Dask](https://dask.org/) and
[Awkward-Array](https://awkward-array.org/). The post has four
sections. The first section uses `dask.dataframe` to introduce the
need for and concept of column projection in Dask. If you are familiar
with column projection in `dask.dataframe`, you can probably skip the
first section.


<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
**Table of Contents**

1. [Projecting columns](#projecting-columns)
2. [Now with dask-awkward](#now-with-dask-awkward)
3. [Necessary columns in dask-awkward](#necessary-columns-in-dask-awkward)
4. [Final thoughts](#final-thoughts)

<!-- markdown-toc end -->


## Projecting columns

Let's start with a simple DataFrame example example where we read a
Parquet dataset from disk into a `pandas.DataFrame` or
`dask.dataframe.DataFrame`. The [Parquet](https://parquet.apache.org/)
specification defines columns, and we can decide which columns to read
from the dataset when instantiating a DataFrame. If we have a dataset
with three columns: "foo", "bar", and "baz", calling

```python
>>> import pandas as pd
>>> df = pd.read_parquet("/path/to/data")
```

will create a DataFrame with all three columns. We can read only "bar"
and "baz" with:

```python
>>> df = pd.read_parquet("/path/to/data", columns=["bar", "baz"])
```

This is beneficial if we know exactly which columns we need to compute
our desired result. In the case above, perhaps we just need the ratio

```python
>>> result = df["baz"] / df["bar"]
```

Reading the entire dataset from disk (that is, including "foo") wastes
compute (more IO) and memory (unnecessary data stored in the
DataFrame).

Pandas is an eager library and each function call is going to run some
compute (like reading data from disk). Pandas cannot look into the
future to see which columns are actually necessary in your program
such that it only reads those columns.

In [dask.dataframe](https://docs.dask.org/en/stable/dataframe.html),
column projection occurs when the task graph detects that some kind of
column selection (via a Python `getitem` call on a DataFrame) has
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
>>> result = df["baz"] / df["bar"]
```

We get a Dask task graph with four steps:

1. Read the Parquet data at "/path/to/data" (no columns specified).
2. Get column "baz" Series.
3. Get column "bar" Series.
4. Divide column "baz" by "bar".

**None of these steps happen until we call**

```python
>>> result.compute()
```

Because of the lazy computation, when we call `.compute()`, Dask can
parse the task graph _before_ computing step one to see that we will
use Python `geitem` calls to grab _only_ columns "baz" and "bar". Side
note: some example column selection `getitem` calls for DataFrames:

- `df["x"]`: grabbing a single column.
- `df[["x", "y"]]`: grabbing multiple columns.
- `df.x`: grabbing a simple column by attribute access.

Back to our example dataset with columns "foo", "bar", and "baz": Dask
will detect that there are `getitem` calls for accessing "bar" and
"baz", so step one in the graph will be **automatically** updated such
that at the `read_parquet` call-site during Dask compute, a `columns=`
argument will be passed to the Pandas function call. The task graph is
rewritten to have step one be:

1. Read the Parquet data at "/path/to/data" (now with the argument
   `columns=["bar", "baz"]`)

The remaining steps will be unchanged. This was simple example of
column projection in `dask.dataframe`. It is one optimization that is
executed when Dask performs a compute. Please [read more about Dask
optimizations in
general](https://docs.dask.org/en/stable/optimize.html) if you are
interested.

## Now with dask-awkward

When we started working on a necessary columns optimization in
dask-awkward, our starting point was the the `getitem`-discovery
technique that we described above. It was a good starting point
because awkward-array uses pandas-like column access for awkward array
**fields**. In awkward-array terms we say records have one of more
fields. One difference between awkward-array and Pandas is that
awkward-array give users the ability to work with arbitrarily nested
data. That nesting can be lists-of-lists or lists-of-records or
lists-of-records where each record can also contain more
lists-of-records or lists-of-lists, and so on.

Let's use the "foo", "bar", and "baz" example again, but now as an
Awkward Array, and actually make it "awkward" with some
non-rectilinearlity. We'll give "foo" and "baz" some sub-fields.
Imagine a dataset that is a list of records like of the form:

```python
[
    {
        "foo": {"x": 1, "y": 2},
        "bar": "yellow",
        "baz": {"a": [7, 8, 9], "b": [1.1, 2.2]},
    },
    {
        "foo": {"x": 9, "y": 8},
        "bar": "orange",
        "baz": {"a": [10], "b": [3.3, 4.4, 5.5, 6.6]},
    },
    ...
]
```

In this example the columns are not just "foo", "bar", and "baz", they
are:

- `foo.x`
- `foo.y`
- `bar`
- `baz.a`
- `baz.b`

Let's say we want to compute `baz.b * foo.x` and this data is stored
in disk in Parquet format:

```python
>>> import awkward as ak
>>> array = ak.read_parquet("/path/to/data")
>>> result = array.baz.b array.foo.x
>>> result.tolist()
[[1.1, 2.2], [29.7, 39.6, 49.5, 59.4], ...]
```

These are all the same in awkward-array, just different access syntax:

```python
>>> array.baz.b * array.foo.x
>>> array["baz"]["b"] * array["foo"]["x"]
>>> array["baz", "b"] * array.foo["x"]
>>> array["baz"].b * array["foo", "x"]
```

Now we arrive at the same scenario we saw above in the Pandas case. We
read more than is necessary.

Since this data is stored in Parquet, we can use a `columns=` argument
to read in exactly what what we need (just like the DataFrame case
above):

```python
>>> import awkward as ak
>>> ak.read_parquet("/path/to/data", columns=["foo.x", "baz.b"])
```

Let's start using
[dask-awkward](https://github.com/dask-contrib/dask-awkward) now.

```python
>>> import dask_awkward as dak
>>> array = dak.read_parquet("/path/to/data")
>>> result = array.baz.b * array.foo.x
```

The task graph steps are:

1. Read data at "/path/to/data"
2. Get field `baz` from top level `array`
3. Get field `b` from the `baz` array
4. Get field `foo` from top level `array`
5. Get field `x` from the `foo` array
6. Multiply output of (3) by output of (5)

The same `getitem`-discovery optimization can _partially_ optimize
this scenario. It discovers `getitem` calls that are direct dependents
of the step in the task graph that does the actual reading from disk.
So in this example, those columns/fields would just be "foo" and
"baz". So we would end up writing rewriting step one as:

```python
>>> dak.read_parquet("/path/to/data", columns=["foo", "baz"]
```

Ok, that's an improvement; **but**, by passing in `columns=["foo",
"baz"]`, we would read the whole set of subfields. That is, "foo.x",
"foo.y", "baz.a", and "baz.b" will be read from disk. Recall that we
only need "foo.x" and "baz.b". This is a bit of a problem. Depending
on the types of these columns, we can be reading a huge amount of
unnecessary data.

## Necessary columns in dask-awkward

We wanted to do better than only selecting and projecting the highest
level fields our Awkward Array. Instead of searching for specific
function calls in the graph before compute time and basing the
optimization on the arguments passed to those functions, we actually
execute the task graph!

Awkward Array has a wonderful feature called typetracer arrays. These
are awkward Arrays that don't actually contain any data buffers, but
retain the structure and types of the awkward arrays that they are
meant to represent. It's like a NumPy dtype on steroids. Almost all
awkward functions and methods work on these typetracer arrays. For
example, if you multiply two typetracer arrays, the result will be
another typetracer array of the correct type and structure, deduced
from the type and structure of the arrays that are being multiplied:


```python
>>> import awkward as ak
>>> a = ak.Array([1.0, 2.0, 3.0])
>>> b = ak.Array([[4, 5], [6], [7, 8, 9]])
>>> a = ak.Array(a.layout.to_typetracer(forget_length=True))
>>> b = ak.Array(b.layout.to_typetracer(forget_length=True))
>>> a
<Array-typetracer [...] type='## * float64'>
>>> b
<Array-typetracer [...] type='## * var * int64'>
>>> a * b
<Array-typetracer [...] type='## * var * float64'>
```

The double hash syntax shows that the typetracer doesn't know the
length of the array (this makes sense, because it only describes
metadata and does not contain any data buffers). Field access on a
typetracer array works the same, accessing the field on a typetracer
array just returns the correct downstream typetracer array.

With this feature, we can run the entire task graph, but **only on
data-less typetracer arrays**. Since there's no real data, the
execution negligible compared to computing on real data from disk. In
our example above, we rewrite step one in the task graph to just be a
typetracer array of the same form of the data in the Parquet file.
That is, an array with all of the known fields organized in the
correct layout. This only requires reading Parquet metadata, not any
of the data buffers. After rewriting the first step in the task graph,
we can just reuse the rest of the graph, and execute! This is possible
because awkward-array functions work on typetracer versions of awkward
Arrays.

The only missing piece is information about which fields get used in
the graph. We grab this information by attaching a mutable typetracer
report object to the first layer of the typetracer based graph. After
executing the typetracer based graph, the report object tells us which
exact fields were touched along the lifetime of the computation. This
is possible because we create a mapping that connects a string label
key to a value that is data-less buffer. When that dataless buffer is
touched during execution of the typetracer only task graph, it's key
is recorded as necessary, the mutable typetracer report tracks this
information.

As of dask-awkward version `2023.3.0` [the magic happens
here](https://github.com/dask-contrib/dask-awkward/blob/2023.3.0/src/dask_awkward/lib/optimize.py#L235),
on line 235 in dask-awkward's `optimize.py` module. We simply call

```python
dask.local.get_sync(typetracer_graph, last_layer)
```

Which computes for the last layer in the typetracer only graph. We are
just using Dask's local scheduler on the (slightly altered) task graph
already stitched together by the user! This is where we are using
dask-awkward to speed up dask-awkward.

Going back to our example result:

```python
>>> result = array.baz.b * array.foo.x
```

In dask-awkward, we can inspect the necessary columns from the input
with
[`dask_awkward.necessary_columns`](https://dask-awkward.readthedocs.io/en/stable/generated/dask_awkward.necessary_columns.html):

```python
>>> dak.necessary_columns(result)
{"read-parquet-abc123": ["foo.x", "baz.b"]}
```

This creates a mapping connecting the input layer name (in the example
above we use "abc123" as shorthand for hash that is appended to all
Dask layer names) in the graph (step one in the example we've been
using) to the columns that need to be read from disk. This is a
utility function in dask-awkward for users to inspect their graph.
When running

```python
>>> result.compute()
```

dask-awkward automatically rewrites step one to use the correct
`columns=["foo.x", "baz.b"]` argument when calling `ak.read_parquet`
at compute time.

## Final thoughts

Two data formats support this optimization today: Parquet and
[ROOT](https://rot.cern). Parquet support is built in to dask-awkward,
while ROOT support is provided by the
[uproot](https://github.com/scikit-hep/uproot5) project and the
`uproot.dask` interface. We still have some work to do. We've been
working through some edge cases where the optimization has been too
greedy. Also, an item on our to-do list is to add support for
optimizing reading JSON data. This will use the discovered necessary
columns to generate a JSON Schema that allows Awkward-Array's JSON
reader to skip whole parts of line delimited JSON files.

This was few months long team effort, and I'd like to thank [Jim
Pivarski](https://github.com/jpivarski) and [Angus
Hollands](https://github.com/agoose77) from the Awkward-Array team for
their input during discussions and their upstream Awkward development
based on our dask-awkward needs. Also thanks to [Lindsey
Gray](https://github.com/lgray) for testing a lot of this work on the
fly. Finally, thanks to my colleague [Martin
Durant](https://github.com/martindurant) for helping in all parts of
dask-awkward development.
