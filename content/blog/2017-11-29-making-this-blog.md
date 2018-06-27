---
layout: post
title: "Making this blog/website"
date: 2017-11-29
---

## Update 2! (June 2018)

Now I'm using Hugo. See [this post]({{<ref
"blog/2018-06-26-move-to-hugo.md" >}}).

## Update! (March 2018)

I'm now using [Netlify](https://www.netlify.com/) to serve this
website with an SSL certificate provided by [Let's
Encrypt](https://letsencrypt.org/).

## Tools

After seeing many well put together technical blogs I finally decided
it was time to take a crack at it while I had some free time over the
recent Thanksgiving holiday. I really enjoy doing everything (outside
of web browsing) in a shell and Emacs, so I didn't want a WYSIWYG
browser based blogging platform. I decided to go with the following
combination:

- Domain from Squarespace.
- Hugo ~~Jekyll~~ site hosted on Netlify ~~GitHub pages~~.
- Let's Encrypt ~~Cloudfare~~ for https://.

## Setting up

I listen to enough NPR and podcasts to have a lifetime of discounts
from Squarespace (if only it wasn't first time purchases only). I
found a domain I liked (using my *nix username, ddavis) with a nice
[TLD](https://en.wikipedia.org/wiki/Top-level_domain) which made for,
in my opinion, a cool blog title. I found ~~[somebody
else](https://github.com/jamesroutley/routley.io)'s
[Jekyll](https://jekyllrb.com/) theme~~ a nice
[Hugo](https://gohugo.io)
[theme](https://themes.gohugo.io/manis-hugo-theme/), hacked it up bit
for myself, and set up a
[repository](https://github.com/drdavis/ddavis.fyi) to be built by
[Netlify](https://www.netlify.com/) ~~[GitHub
pages](https://pages.github.com).~~

After getting a functional Jekyll build using Netlify I configured the
site to use Let's Encrypt. ~~in GitHub pages~~, ~~I headed over to
~~[Cloudfare](https://www.cloudflare.com/) to set up my new domain and
site with a free SSL certificate. Their
[instructions](https://blog.cloudflare.com/secure-and-fast-github-pages-with-cloudflare/)
were very easy to follow.~~

And that's it. Now updating my site is a git commit+push away.
