---
layout: page
title: 
tagline: Supporting tagline
description: "记录工作生活大小事，推崇极简主义的个人博客"
tag: [Arch Linux，Cisco，网络技术，生活]
---
{% include JB/setup %}


<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

