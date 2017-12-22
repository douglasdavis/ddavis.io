---
layout: post
title:  "ROOT without ROOT"
disqustitle: "rootnoroot"
comments: false
---

------

The high energy physics community has used [ROOT](https://root.cern/)
([GitHub project](https://github.com/root-project/root)) for over 20
years now. It's a very large, monolithic set of libraries packaged up
with a C++ interpreter called [cling](https://root.cern.ch/cling)
([GitHub project](https://github.com/root-project/cling)). There are a
few examples of [ROOT
criticism](http://insectnation.org/articles/problems-with-root.html)
out in the wild -- the title of this post may suggest a critical post,
but it's not!

I'll start discussing using ROOT without ROOT with an
endorsement. ROOT is awesome for storing C++ classes on disk. As an
example: In ATLAS we use C++ classes to represent different things in
our detector and we need to store that data to disk. We have classes
for events as a whole, hits in the detector, a whole reconstructed
particle, etc. ROOT is great for storing this in an intuitive way, for
example: particles live in containers owned by an event, hits live in
a container owned by a track, whole reconstructed particles have an
"Element Link" (a pointer on disk) to a track associated with it, etc.

## The problem

ROOT Is a monolithic beast. It's a lot to carry around if all you need
is to look at a few numbers stored in a ROOT file. It takes a while to
build the entire library and interpreter. The ROOT team distributes
some binaries, and some package managers provide binaries or a way to
build locally (e.g. the [Arch User's
Repository](https://aur.archlinux.org/)); but let's face it, for
beginners and quick tasks that's not always a great solution.

Then to actually look at your data you end up writing a C++ "macro"
(meant to be processed by ROOT's C++ interpreter, cling), or you write
a proper executable, compile, link, and run.

## The old solution

If your ROOT build was aware of a python installation during the build
process, you can end up with PyROOT - ROOT's builtin python
bindings. PyROOT basically allows you to write C++ style code in
python to talk to ROOT objects. That's not even the old solution I'm
about to mention. `root_numpy` is what I'd consider the old solution
here -- it's a python library accelerated with Cython which turns the
C style arrays stored in ROOT files into numpy arrays. It can also be
installed with `pip`! Unfortunately, it requires a ROOT installation
(because it requires `import ROOT`).

## The solution

Now enters `uproot`. This awesome new library requires _only_
numpy. Being able to interact with ROOT files is as easy as

```
 pip install uproot
 python
 >>> import uproot
 >>> file = uproot.open(myfile.root')
```

## In action

A few days ago I needed to throw together a quick histogram to explain
a task to a colleague. The task required just a bit of information
about some hits along a track. Given the structure of our data format
stored in ROOT files, I would need to do something like this (in _kind
of_ pseudo C++ code, this is very similar to ATLAS code, but with a
few made up function names):

```cpp
 for ( auto& event : eventContainer() ) {
   // grab particle container
   auto particleContainer = event->getParticleContainer();
   // loop over particles
   for ( auto& particle : particleContainer ) {
     // get link to track and make sure valid
     auto trackLink = getAssociatedTrackLink(particle);
     if ( trackLink.isValid() ) {
       // dereference link to get actual object (the track)
       auto track = *trackLink;
       // get link to hit container and make sure valid
       auto hitContainerLink = getAssociatedHitsLink(track);
       if ( hitContainerLink.isValid() ) {
         auto hitContainer = *hitContainerLink;
         // loop over container
         for ( auto& hit : hitContainer ) {
           // get dynamically set properties of the hit and finally use them
           auto hitFoo = hit->getAuxiliaryData("Foo");
           auto hitBar = hit->getAuxiliaryData("Bar");
           if ( hitBar == 42 ) {
             fooHistogram->Fill(hitFoo);
           }
         }
       }
     }
   }
 }
```
