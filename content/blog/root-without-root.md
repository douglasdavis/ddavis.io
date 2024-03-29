+++
title = "ROOT analysis without ROOT"
date = 2018-02-02
draft = false
[taxonomies]
tags = ["hep", "numpy", "cpp", "python"]
+++

The high energy physics community has used [ROOT](https://root.cern/)
since the late 90's. It's a very large and monolithic set of libraries
packaged up with a C++ interpreter called
[Cling](https://root.cern.ch/cling). ROOT's strength (in my opinion)
is its ability to serialize C++ objects to disk in binary format (you
can read all about it
[here](https://root.cern.ch/root/htmldoc/guides/users-guide/InputOutput.html)).
This is perfect for HEP. We have classes for events as a whole,
classes for hits in the detector, classes for whole reconstructed
particles, etc. ROOT is great for storing this in an intuitive way,
for example: particles live in containers owned by an event, hits live
in a container owned by a track, whole reconstructed particles have an
"Element Link" (a class to act as a pointer on disk) to a track
associated with it, etc.


## The problem {#the-problem}

ROOT is a monolithic beast. It's a lot to carry around if all one
needs to do is look at a few numbers stored in a ROOT file. It takes a
while to build the entire library (and the packaged interpreter). The
ROOT team distributes some binaries, and some package managers provide
binaries or a way to build locally (e.g. the [Arch User's
Repository](https://aur.archlinux.org/))... but for beginners and
quick tasks that's not always a great solution[^1].

Then, to actually look at one's data a C++ "macro" has be be written
(not a compiler preprocessor macro, this is something that is meant to
be processed by ROOT's C++ interpreter, cling); or, one writes a
proper executable, compile it, link it, and run it. This C++ code can
be verbose and full of boilerplate (especially for reading ROOT files,
where one has to connect C++ variables to ROOT "branches", one line at
a time[^2]).


## The old solution {#the-old-solution}

If a ROOT build was aware of a python installation during the build
process, one can end up with PyROOT - ROOT's builtin python bindings.
PyROOT basically allows writing C++ style code in python to talk to
ROOT objects. That's not even the old solution I'm about to mention.
[root-numpy](https://github.com/scikit-hep/root_numpy) is what I'd
consider the _old_ solution -- it's a python library accelerated with
Cython which turns the C style arrays stored in ROOT files into numpy
arrays. It can also be installed with pip. Unfortunately, it requires
a ROOT installation (because it requires `import ROOT`).


## The solution {#the-solution}

Now enter [uproot](https://github.com/scikit-hep/uproot). This awesome
new library is pure Python and does not require a ROOT installation.
We can interact with ROOT files is as easy as:

```
$ pip install uproot
$ python
>>> import uproot
>>> file = uproot.open("myfile.root")
```

uproot has knowledge of ROOT's binary format implemented _completely
in python_. No ROOT installation required.


## In action {#in-action}

A few days ago I needed to throw together a quick histogram to explain
a task to a colleague. The task required just a bit of information
about some hits along a track. Given the structure of our data format
stored in ROOT files, I would need to do something like this cascade
of data retrieval (in _kind of_ pseudo C++ code, this is very similar
to [ATLAS code](https://gitlab.cern.ch/atlas/athena/), but with a few
made up function names):

```C++
// some histogram object that we're going to fill with data
ns::Histogram fooHistogram(20, 0.0, 100.0);

for (const auto& event : eventContainer()) {
  // grab particle container
  const ns::ParticleContainer* particleContainer = event->getParticleContainer();
  // loop over particles
  for (const auto& particle : particleContainer) {
    // get link to track and make sure valid
    auto trackLink = getAssociatedTrackLink(particle);
    if (!trackLink.isValid()) {
      continue;
    }
    // dereference link to get actual object (the track pointer)
    const ns::Track* track = *trackLink;
    // get link to hit container and make sure valid
    auto hitContainerLink = getAssociatedHitsLink(track);
    if (!hitContainerLink.isValid()) {
      continue;
    }
    const ns::HitContainer* hitContainer = *hitContainerLink;
    // loop over container
    for (const auto& hit : hitContainer) {
      // get dynamically set properties of the hit and finally use them
      float hitFoo = hit->getAuxiliaryData<float>("foo");
      int hitBar = hit->getAuxiliaryData<int>("bar");
      if (hitBar == 42) {
        fooHistogram.Fill(hitFoo);
      }
    }
  }
}

fooHistogram.Draw(/* some options */);
```

In python, with uproot, if I know the naming convention for the hit
container, I can simply write:

```python
import uproot
import matplotlib.pyplot as plt

datatree = uproot.open("myfile.root")["data"]
bar = datatree.array("innerdetector.hits.auxdata.bar")
foo = datatree.array("innerdetector.hits.auxdata.foo")
selected_foo = foo[bar == 42]

plt.hist(selected_foo, bins=20)
plt.show()
```

The python code is very simple and to the point, it's fast because the
binary format is being read directly into `numpy` arrays[^3].

There is _absolutely_ a place for the C++ code. If I wanted to apply a
complex set of requirements to select different objects _above_ the
hit level (but based on hit properties), I need this structure. If we
had a perfectly columnar data format (each event as a row in a table
and a column for every feature), the hit information would be
duplicated in multiple places because a low level hit may be
associated with multiple higher level objects. Given our many
petabytes of data, this is not feasible. This is where the "links"
come in (the pointers on disk that tell a track where the associated
hits are).

In this simple case, I didn't care about selecting hits based on any
other information except another (simply) accessible hit property.

To wrap up: it's nice to have (a) an isolated python library for
accessing data stored in ROOT and (b) _options_ for selecting tools to
analyze data.

---

[^1]: Update summer 2019: ROOT is now available as a
    [conda-forge](https://conda-forge.org/) package, providing a very
    easy installation method.

[^2]: As of ROOT version 6.14 (released June 2018) there is a new
    feature allowing tree analysis using functional chains with the
    [`RDataFrame`](https://root.cern.ch/doc/master/classROOT_1_1RDataFrame.html)
    class.

[^3]: In some special situations (e.g. reading a column of
    `std::vector` objects into a jagged array) the implementation is
    accelerated with [numba](https://numba.pydata.org/) (if
    installed).
