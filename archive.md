---
layout: default
title: Archive Posts
id: Archive
permalink: /archive/
---

<div>
<h1>{{ page.title }}</h1>

  {% for post in site.posts %}{% if post.title != null %}
  <div>
    <span style="float: left;">
      <a href="{{ post.url }}">{{ post.title }}</a>
    </span>

    <span style="float: right;">
      {{ post.date | date_to_string }}
    </span>
  </div>
  <div style="clear: both;">
  </div>
  {% endif %}{% endfor %}

</div>
