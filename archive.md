---
layout: page
title: Статьи
permalink: /archive/
---

{% for post in site.posts %}
 [ {{ post.title }} ]({{ post.url }})
{% endfor %}
