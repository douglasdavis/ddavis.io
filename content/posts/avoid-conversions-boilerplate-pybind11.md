+++
title = "Avoiding conversions & boilerplate in pybind11"
date = 2021-02-11
tags = ["cpp", "python", "numpy"]
draft = false
+++

The goals of [pygram11](https://github.com/douglasdavis/pygram11) include:

-   providing the fastest possible histogram calculations.
-   supporting uncertainties on weighted histograms.
-   supporting multiple weight variations in a single histogramming
    procedure.

To first order, OpenMP is leveraged to execute parallel loops for
calculating histograms of large input data. This post focuses on a
second order performance consideration: avoiding potentially expensive
conversions (while supporting different data and weight array types).

Early versions of pygram11 (up to version 0.10.3) supported input
arrays of any type, but in the backend we supported histogramming
calculations only on 32- and 64-bit floating point inputs (for both
the data and the weights). If a non-floating point typed array was
passed (as either the data input or weights input), we converted the
incompatibly typed arrays to floating point arrays and passed the
converted data to the backend C++ functions.

The backend C++ code has always been generic (implemented with
templated functions). An example function and [pybind11](https://github.com/pybind/pybind11) binding was of
this form:

```cpp
template <typename XType, typename WType>
py::tuple f1dw(py::array_t<XType> x, py::array_t<WType> w,
               py::ssize_t nbins, double xmin, double xmax) {
  py::array_t<WType> values(nbins);
  py::array_t<WType> variances(nbins);
  // ...
  // do calculations/pass arrays to other C++ code
  // ...
  return py::make_tuple(values, variances);
}

PYBIND11_MODULE(_backend, m) {
  m.def("_f1dw", &f1dw<double, double>);
  m.def("_f1dw", &f1dw<float, double>);
  m.def("_f1dw", &f1dw<double, float>);
  m.def("_f1dw", &f1dw<float, float>);
}
```

This setup supports `np.float32` (C++ float) and `np.float64` (C++
double) input and weights via a single templated function that is used
by four bindings. This was the implementation before version 0.11.0 of
pygram11.

Now let's look at the example of a two-dimensional weighted histogram
function signature:

```cpp
template <typename XType, typename YType, typename WType>
py::tuple f2dw(py::array_t<XType> x,
               py::array_t<YType> y,
               py::array_t<WType> w,
               py::ssize_t xbins, double xmin, double xmax,
               py::ssize_t ybins, double ymin, double ymax) {
  // ... implementation
}

PYBIND11_MODULE(_backend, m) {
  m.def("_f2dw", &f2dw<double, double, double>);
}
```

Let's say we want to support 32-bit and 64-bit floating, integer, and
unsigned integer input, along with 64-bit and 32-bit floating point
weights. That's six types for both `x` and `y`, and two types for `w`;
that's ****72 total overloads****.

We can lean on some template metaprogramming to make a clean (low
boilerplate) and extendable implementation while achieving the goal of
explicitly avoiding conversions.

We'll use [boost::mp11](https://github.com/boostorg/mp11) for some help. We create a type list for the
possible data types (`pg_Ts`) and one for the possible weight types
(`pg_Ws`). The `boost::mp11` library provides the `mp_product`
metafunction to generate all possible combinations of its template
parameters at compile time.

```cpp
using boost::mp11::mp_product;

template <typename... Ts>
struct type_list {};

// all data types
using pg_Ts = type_list<double, int64_t, uint64_t, float, int32_t, uint32_t>;
// all weight types
using pg_Ws = type_list<double, float>;
// all combinations of data types and weight types
using pg_Ts_and_Ws = mp_product<type_list, pg_Ts, pg_Ws>;
// all combinations of data types and data types
using pg_T_pairs = mp_product<type_list, pg_Ts, pg_Ts>;
// all combinations of data types, data types, and weight types
using pg_T_pairs_and_Ws = mp_product<type_list, pg_Ts, pg_Ts, pg_Ws>;
```

Let's think about our new types a bit:

-   `pg_Ts_and_Ws` is made up of pairs of data types and weight types:
    -   `type_list<double, double>`, (1st)
    -   `type_list<int64_t, double>`, (2nd)
    -   ...
    -   `type_list<uint32_t, float>` (12th)
-   `pg_Ts_and_Ts` is made up of pairs of data types:
    -   `type_list<double, double>`, (1st)
    -   ...
    -   `type_list<uint32_t, uint32_t>` (36th)
-   `pg_T_pairs_and_Ws` is made up of data types (x2), and weight types:
    -   `type_list<double, double, double>` (1st)
    -   ...
    -   `type_list<uint32_t, uint32_t, float>` (72nd)

We can write some small functions to inject into a `py::module_`
object a type list of each of these three forms:

```cpp
// to use "x"_a instead of py::arg("x")
using namespace pybind11::literals;

/// inject a data type and weight type pair
template <typename Tx, typename Tw>
void f1(py::module_& m, const type_list<Tx, Tw>&) {
  m.def("_f1dw", &f1dw<Tx, Tw>,
        "x"_a.noconvert(), "w"_a.noconvert());
}

/// inject and data type and data type pair
template <typename Tx, typename Ty>
void f2(py::module_& m, const type_list<Tx, Ty>&) {
  m.def("_f2d", &f2d<Tx, Ty>,
        "x"_a.noconvert(), "y"_a.noconvert());

}

/// inject a data type, data type, and weight type triplet
template <typename Tx, typename Ty, typename Tw>
void f3(py::module_& m, const type_list<Tx, Ty, Tw>&) {
  m.def("_f2dw", &f2dw<Tx, Ty, Tw>,
        "x"_a.noconvert(), "y"_a.noconvert(), "w"_a.noconvert());
}
```

And then use `boost::mp11::mp_for_each` to inject for each of our type
combinations:

```cpp
PYBIND11_MODULE(module_name, m) {
  using boost::mp11::mp_for_each;
  mp_for_each<pg_Ts_and_Ws>([&](const auto& Ts) { f1(m, Ts); });
  mp_for_each<pg_Ts_and_Ts>([&](const auto& Ts) { f2(m, Ts); });
  mp_for_each<pg_T_pairs_and_Ws>([&](const auto& Ts) { f3(m, Ts); });
  // first function will have Ts == type_pair<T1, T2>
  // second will also have Ts == type_pair<T1, T2>
  // third will be Ts == type_pair<T1, T2, T3>
}
```

These three calls of `mp_for_each` will provide 12 + 36 + 72 overloads
to the `py::module_` instance. One can imagine making an addition to
the supported types of the library: instead of adding multiple new
function calls to cover all possible combinations, we just add it to a
type list and the template metaprogramming takes care of generating
all possible overloads.

This code isn't an exact copy of what is in pygram11 version 0.11.0,
but it's quite close. Checkout the
[\_backend.cpp](<https://github.com/douglasdavis/pygram11/blob/0.11.0/src/%5Fbackend.cpp#L1424-L1499>)
file.
