---
layout: default
title: Doug Davis | Blog
---

<a href="/">Home</a>

# Blog

{% for post in site.categories.content %}
- {{ post.date | date: "%Y-%m-%d" }} - [{{ post.title }}]({{ post.url }}) {% endfor %}

[Subscribe with rss](/feed.xml)

{% include post_footer.html %}
