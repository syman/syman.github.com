---
layout: default
title: 
tagline: 
---

###最新15条:
<ul class="posts">
  {% for post in site.posts limit 25%}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
  <li><span>28 Jun 2013</span> &raquo; <a href="_posts/index.html">Openstack 培训</a></li>
</ul>