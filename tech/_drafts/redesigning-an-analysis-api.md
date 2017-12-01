---
layout: post
title: "Redesigning an Analysis API"
---
------
The [ATLAS](https://atlas.cern) detector is basically an onion - it
has quite a few layers, or subdetectors. The first subdetector is the
[Inner Detector](https://atlas.cern/discover/detector/inner-detector)
(ID). The ID itself is made of three subsubdetectors. I happen to work
in the software group associated with the outermost detector in the
ID, the Transition Radiation Tracker (TRT). The TRT tracks charged
particles as they leave evidence of their presence by ionizing the gas
in our straws. We also take a crack at identifying it's particular
branch brand. I develop and maintain an API designed for analyzing
particle tracks through the TRT the data format designed for ATLAS.

ATLAS recently adopted [CMake](https://cmake.org/) as our build system
and [git](https://git-scm.com/) as our version control system (VCS). I
was happy that we made these changes (we previously used a combination
of home-grown tools for building - CMT and "RootCore" - and SVN for
VCS). This was a major change for all developers and users of ATLAS
software. The process took months and months for everything to move
over. As a member of the TRT SW group and developer of the existing
analysis tools that we had, I watched closely as the ecosystem shifted
and realized our main analysis tool needed some work. I decided it was
a good time to redesign the API.

Along the way I learned some stuff that I think went a long way
towards first enjoying writing a new API for physics analysis, and
second, making me a better software developer. I thought it'd be nice
to write about some.

## Always be gradua... I mean documenting

My two favorite programming languages, C++ and Python, both have some
great inline documentation tools:
[Doxygen](http://www.stack.nl/~dimitri/doxygen/) and
[docstrings](https://www.python.org/dev/peps/pep-0257/), respectively.
I have some experience with both, but I had not completely committed
to _thoroughly_ documenting my software the way I did with this
project. I attended a software meeting at CERN once where I heard the
phrase: "always be documenting," I laughed because it made me think of
the phrase that I hear among my grad student friends: "always be
graduating." The connection helped me remember it.

ATLAS is a big C++ customer. Almost all analysis software in ATLAS has
to at least start in C++; that's what most of our analysis framework
is written in. I made a rule for myself - If you make a class or a
function - don't move on until you give it at least a doxygen
one-liner. I also made sure to spend some time going back for details.

The ATLAS data format relies on the ability to "stick on" auxiliary
data to physics objects. Here's a helper function, documented doxygen
style, that makes it easy to grab a popular piece of aux data.

```cpp
/// return the track occupancy, if available. return -1 if unavailable
float trackOccupancy(const xAOD::TrackParticle* track);
```

That's pretty good, seems easy to understand, but we can do better:

```cpp
/// return the track occupancy, if available. return -1 if unavailable
/**
 *  This is a helper function that wraps the process of retrieving the
 *  auxiliary data "TRTTrackOccupancy from an xAOD::TrackParticle in
 *  one line.
 *
 *  @param track pointer to an xAOD::TrackParticle object.
 *  @return the "TRTTrackOccupancy" auxiliary data (-1 if unavailabe)
 */
 float trackOccupancy(const xAOD::TrackParticle* track);
 ```
 
 The more descriptive documentation gives a user more detail about
 _why_ they might be getting that -1 returned.

Writing the details also helps you find ways to possibly improve the
implementation. If you write what you're actually trying to accomplish
with a function - you might realize there's a better way to do
it. Doing it early helped to organize my API as well. For example, I
started grouping functions together that were related. Needless to
say, like with most things in life, doing it early saves you time
later. These are all things that can be done at some point in the life
of a piece of software, but it only helped to be cognizant of it from
line 0.

Finally, write a short to medium length tutorial, orthogonal to inline
documentation, about how to start using your API. There's great stuff
out there for this; I decided to use
[GitBook](https://www.gitbook.com/).

# TODO

## CI

I'm new to [Continuous
integration](https://en.wikipedia.org/wiki/Continuous_integration)
(CI), but with the existence of `.travis.yml` and `.gitlab-ci.yml`
files in so many repositories I get the feeling that it's easier than
ever to actually do it these days. The move to git was great for this,
because CERN uses [GitLab](https://gitlab.com/) for hosting git
repositories, and GitLab's builtin CI is pretty nice.

## Be your first user

## Be influenced

If you're writing in any programming language, you're using something
that has likely been used to write a whole bunch of other things, and
there's a good chance you might be using a library or two (or 10 or
12). I was using some existing ATLAS libraries and looking at both the
code and documentation for those libraries helped me get more out of
them and give me ideas to make my API (and my documentation), in my
opinion, better.

I'm one of a large number of people writing a blog post about writing
an API - so there are plenty of opportunities to learn from [some of
the
greats](https://blog.keras.io/user-experience-design-for-apis.html).

## So what is it?

The software I've been talking about this whole time is:
[TRTFramework](https://gitlab.cern.ch/atlas-trt-software/TRTFramework)
