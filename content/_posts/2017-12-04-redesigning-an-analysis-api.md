---
layout: post
title: "Redesigning an Analysis API"
date: 2017-12-04
disqustitle: "apiredesign"
---

------

The [ATLAS](https://atlas.cern) detector is basically an onion - it
has quite a few layers, or subdetectors. The first subdetector is the
[Inner Detector](https://atlas.cern/discover/detector/inner-detector)
(ID). The ID itself is made of three subsubdetectors. I work in the
software group associated with the outermost detector in the ID, the
Transition Radiation Tracker (TRT). The TRT tracks charged particles
as they leave evidence of their presence by ionizing the gas in our
straws. We also take a crack at identifying its particular type. I
develop and maintain an API designed for analyzing particle tracks
through the TRT using the data format designed for ATLAS.

ATLAS recently adopted [CMake](https://cmake.org/) as our build system
and [git](https://git-scm.com/) as our version control system (VCS). I
was happy that we made these changes (we previously used a combination
of home-grown tools for building - CMT and "RootCore" - and SVN for
VCS). This was a major change for all developers and users of ATLAS
software. The process took months and months for everything to move
over. As a member of the TRT SW group and developer of the existing
analysis tools that we had, I watched closely as the ecosystem and
recommendations from our "Analysis Software Group" shifted. At the
same time, I had started working on a machine learning project that
required extracting the data in a way that wasn't possible with the
existing API. I decided that, with that combinations of motivations,
it was a good time to redesign the API.

Along the way I learned some stuff that I think went a long way
towards, first, enjoying writing a new API for physics analysis, and
second, making me a better software developer. I thought it'd be nice
to write about some.

## Find inspiration

If you're writing in any programming language, you're using something
that has likely been used to write a whole bunch of other things, and
there's a good chance you're using a library or two (or 10 or 12). I
was using some existing ATLAS libraries; looking beyond the
documentation and at the code helped me get more out of them and give
me ideas to make my API (and my documentation), in my opinion, better.

In my case, I also took inspiration from the API I was
redesigning. There were a number of functionalities that I wanted to
keep, but I made sure to not copy and paste any of the code that was
simple and sticking around. Typing it out made me think about what I
was typing and how it could possibly be improved.

I'm one of a large number of people writing a blog post about writing
an API - so there are plenty of opportunities to learn from [some of
the
greats](https://blog.keras.io/user-experience-design-for-apis.html).

## Always be gradua... I mean documenting

My two favorite programming languages, C++ and Python, both have some
great inline documentation tools:
[Doxygen](http://www.stack.nl/~dimitri/doxygen/) and
[docstrings](https://www.python.org/dev/peps/pep-0257/), respectively.
I have some experience with both, but I had not completely committed
to _thoroughly_ documenting my software the way I did with this
project. I attended a software meeting at CERN once where I heard the
phrase "always be documenting," I laughed because it made me think of
the phrase that I hear among my grad student friends: "always be
graduating." The connection helped me remember it.

ATLAS is a big C++ customer. Almost all analysis software in ATLAS has
to at least start in C++; that's what most of our analysis framework
is written in. I made a rule for myself - If you make a class or a
function - don't move on until you give it at least a doxygen
one-liner. I also made sure to spend some time going back for details.

The TRT has different types of hits along the track, one is
"precision", the precision hit fraction (PHF) is a popular variable of
interest; here's a documented function to calculate it:

```cpp
 /// return the precision hit fraction along the track, -1 if hits unavailable.
 float trackPHF(const xAOD::TrackParticle* track);
```

That's pretty good, seems easy to understand, but we can do better:

```cpp
 /// return the precision hit fraction along the track, -1 if hits unavailable.
 /**
  *  This is a helper function that wraps the process of looping over the
  *  hits to calculate the fraction of precision hits in one line. The
  *  auxiliary data "msosLink" (container of hits) must be available (along
  *  with the biased and unbiased residual and pull variables) on the
  *  xAOD::TrackParticle object, if not, then return -1.
  *
  *  @param track pointer to an xAOD::TrackParticle object.
  *  @return the precision hit fraction (-1 if hit container unavailable)
  */
  float trackPHF(const xAOD::TrackParticle* track);
 ```
 
Let that user know, in detail, why -1 is returned.

Writing the details also helps you find ways to possibly improve the
implementation. If you write what you're actually trying to accomplish
with a function - you might realize there's a better way to do
it. Doing it early helped to organize my API as well. 

Finally, write a short to medium length tutorial, orthogonal to inline
documentation, about how to start using your API. There's great stuff
out there for this; I decided to use
[GitBook](https://www.gitbook.com/). I wish I had started that sooner
than I did.

## Be your first user

Having a project to develop in parallel with the API (that is of
course using it) helped me a lot. First, it helped me think about what
should be part of the generic API and what should be left to the
user. I implemented a number of functions on one side, but at the end
of the day moved it over.

Being your first user also helps the developer version of you see how
someone would use the API in ways that developer you might not see.

## CI

I'm new to [Continuous
integration](https://en.wikipedia.org/wiki/Continuous_integration)
(CI), but with the existence of `.travis.yml` and `.gitlab-ci.yml`
files in so many repositories I get the feeling that it's easier than
ever to actually do it these days. The move to git was great for this,
because CERN uses [GitLab](https://gitlab.com/) for hosting git
repositories, and GitLab's builtin CI is pretty nice. 

As of right now, while the new API is in alpha, I set up CI to make
sure that the library compiles (I do this by compiling it independent
of anything and also alongside a project that's using it). It's as
simple as defining a [Docker](https://www.docker.com/) image to use
and defining a few shell commands to run. I know that in the future
I'll always make sure to start projects with at least a CI system
building the code. Next on my to-do list is implementing some unit
tests.

## So what is it?

The software I've been talking about this whole time is:
[TRTFramework](https://gitlab.cern.ch/atlas-trt-software/TRTFramework),
which is an API originally developed by a collaborator who has since
moved on from the TRT, before the massive ATLAS SW migration of
2016/2017. The SW group as a whole took it over, and as deputy
coordinator in the group I took over maintaining it -- we now have
version 2.
