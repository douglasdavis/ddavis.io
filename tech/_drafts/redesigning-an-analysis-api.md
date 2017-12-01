---
layout: post
title: "Redesigning an Analysis API"
date: 2017-11-30
---

The [ATLAS](https://atlas.cern) detector is basically an onion - it
has quite a few layers, or subdetectors. The first subdetector is the
[Inner Detector](https://atlas.cern/discover/detector/inner-detector)
(ID). The ID itself is made of three subsubdetectors. I happen to work
in the software group associated with the outermost detector in the
ID, the Transition Radiation Tracker (TRT). I develop and maintain an
API designed for analyzing particle tracks through the TRT the data
format designed for ATLAS.

ATLAS recently adopted [CMake](https://cmake.org/) as our build system
and git as our VCS/SCM. I was happy to make the switch (from a
combination of home-grown tools CMT and "RootCore" for building and
SVN for VCS/SCM). This was a major change for all developers and users
of ATLAS software. The process took months and months for everything
to move over. As a member of the TRT SW group and developer of the
existing analysis tools that we had, I watched closely as the
ecosystem shifted and realized our main analysis tool needed some
work. I decided it was a good time to redesign the API.

Along the way, I forced upon myself some practices that I think went a
long way towards enjoying writing a new API for physics analysis. I
thought it'd be nice to write about some.

# Immediate Documentation

My two favorite programming languages, C++ and Python, both have some
great inline documentation tools:
[Doxygen](http://www.stack.nl/~dimitri/doxygen/) and
[docstrings](https://www.python.org/dev/peps/pep-0257/), respectively.
I have some experience with both, but I had not completely committed
to _thoroughly_ documenting my code the way I did
