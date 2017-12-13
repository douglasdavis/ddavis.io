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
      auto track = *track;
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
