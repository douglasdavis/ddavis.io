---
layout: default
title: Doug Davis | Home
---

# Doug Davis, for your information

Graduate student and Ph.D. candidate in Experimental Elementary
Particle Physics at [Duke University](https://www.duke.edu/). Member
of the [ATLAS collaboration](https://atlas.cern/) at
[CERN](https://home.cern/).

- <i class="fa fa-github" aria-hidden="true"></i> [GitHub](https://github.com/drdavis)
- <i class="fa fa-gitlab" aria-hidden="true"></i> [CERN GitLab](https://gitlab.cern.ch/ddavis)
- <i class="fa fa-linkedin" aria-hidden="true"></i> [LinkedIn](https://www.linkedin.com/in/douglasrdavis)
- <i class="fa fa-twitter" aria-hidden="true"></i> [Twitter](https://twitter.com/_ddavis_)
- <i class="fa fa-file-pdf-o" aria-hidden="true"></i> [Curriculum Vitae](/assets/pdf/cv.pdf)

## Blog

[All posts](/blog.html)
{% for post in site.categories.content limit: 3 %}
- {{ post.date | date: "%Y-%m-%d" }} - [{{ post.title }}]({{ post.url }}) {% endfor %}

## About

My favorite parts of experimental particle physics are writing
software and analyzing data. My blog posts are (mostly) born from
technical things I encounter along the way to getting that Ph.D.

I'm currently in my fourth year at Duke where I work in the ATLAS
group as deputy coordinator in the transition radiation tracker (TRT)
software group and as an analyzer in the single top working group.  My
thesis analysis will study the cross section of the production of a
top quark in association with a W boson.
