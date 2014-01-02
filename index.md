---
layout: page
title: 
tagline: Supporting tagline
description: "记录工作生活大小事，推崇极简主义的个人博客"
tag: [Arch Linux，Cisco，网络技术，生活]
---
{% include JB/setup %}

<ul class="posts">
{% for post in site.posts  %}  
  <li><article><a href="{{ site.url }}{{ post.url }}">{{ post.title }} <span class="entry-date"><time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%B %d, %Y" }}</time></span></a></article></li>
{% endfor %}
</ul>
