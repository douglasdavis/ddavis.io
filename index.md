---
layout: default
title: Doug Davis | Home
---

# Doug Davis, for your information

Graduate student and Ph.D. candidate in Experimental Elementary
Particle Physics at [Duke University](https://www.duke.edu/). Member
of the [ATLAS collaboration](https://atlas.cern/) at
[CERN](https://home.cern/).

- [GitHub](https://github.com/drdavis), [CERN GitLab](https://gitlab.cern.ch/ddavis)
- [LinkedIn](#)
- [Twitter](https://twitter.com/_ddavis_)
- [Curriculum Vitae](#) (pdf)

## Blog

[All posts](/blog.html)
{% for post in site.categories.tech limit: 3 %}
- `{{ post.date | date: "%Y-%m-%d" }}` - [{{ post.title }}]({{ post.url }}) {% endfor %}

## About

My favorite parts of experimental particle physics are writing
software and analyzing data. My blog posts are born from technical
things I encounter along the way to getting that Ph.D.

I'm currently in my fourth year at Duke where I work in the ATLAS
group as deputy coordinator in the transition radiation tracker (TRT)
software group and as an analyzer in the single top working group.  My
thesis analysis will study the cross section of the production of a
top quark in association with a W boson.

[//]: #(## Selected Projects)

[//]: #(- [TRTFramework](https://gitlab.cern.ch/atlas-trt-software/TRTFramework),)
[//]: #(  an analysis framework for TRT studies.)
[//]: #(- [TopLoop](https://github.com/drdavis/TopLoop), a framework for)
[//]: #(  analyzing top ntuples (ATLAS top group data format).)
