---
layout: default
title: Blog Archives
---

# {{ page.title }}

{% for post in site.posts %}
<h3>
  <small><i class="fa fa-calendar"></i>&nbsp; {{ post.date | date: "%b %d, %Y" }}</small><br />
  <a href="{{ post.url }}">{{ post.title }}</a>
</h3>
{% endfor %}
