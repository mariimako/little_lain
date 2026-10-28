---
title: Blog
layout: default
nav_order: 2
---
# Blog Posts

I write a very rough, non-professional blog where I motivate myself to note interesting things I learn and share them here.

{% assign posts = site.pages | where: "dir", "/blog/" | where_exp: "p", "p.name != 'blogs.md'" | sort: "date" | reverse %}
{% for post in posts %}
- [{{ post.title }}]({{ post.url | relative_url }})  
  <small>{{ post.date | date: "%B %d, %Y" }}</small>
{% endfor %}
