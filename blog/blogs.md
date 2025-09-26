---
title: Blog
layout: default
nav_order: 2
---

# Blog Posts

I am writing a very rough, non professional blog where I motivate myself in noting interesting things I learn and share it here. 


{% for post in site.posts %}
- [{{ post.title }}]({{ post.url | relative_url }})  
  <small>{{ post.date | date: "%B %d, %Y" }}</small>
{% endfor %}
