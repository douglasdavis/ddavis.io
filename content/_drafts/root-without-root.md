---
layout: post
title:  "ROOT without ROOT"
date: 2017-12-09
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
out in the wild -- the title of this post may suggest its a critical
post, but it's not!

I'll start discussing using ROOT without ROOT with an
endorsement. ROOT is awesome for storing C++ classes on disk. As an
example: we utilize this heavily in the ATLAS software ecosystem. We
use C++ classes to represent different things in our detector: Events
as a whole, hits in the detector, a whole reconstructed particle,
etc. ROOT is great for storing this in an intuitive way, for example:
particles live in containers owned by an event, hits live in a
container owned by a track, whole reconstructed particles have an
"Element Link" (a pointer on disk) to a track associated with it, etc.

```cpp
auto event = getEvent(); // grab event
auto particleContainer = event->getParticleContainer(); // grab particle container
for ( auto& particle : particleContainer ) { // loop over particles
  auto trackLink = getAssociatedTrackLink(particle); // get link to track
  if ( trackLink.isValid() ) { // make sure link is valid
    auto track = *track; // dereference link to get actual object (the track)
    auto hitContainerLink = getAssociatedHitsLink(track); // get link to hit container
    if ( hitContainerLink.isValid() ) { // make sure its valid
      auto hitContainer = *hitContainerLink; // dereference
      for ( auto& hit : hitContainer ) { // loop over container
        auto hitFoo = hit->getAuxiliaryData("Foo"); // get dynamically set property of the hit
        myHistogram->Fill(hitFoo); // finally use it to fill a histogram
      }
    }
  }
}
```
