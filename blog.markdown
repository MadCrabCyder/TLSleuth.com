---
layout: page
title: Blog
permalink: /blog/
---

{% assign posts_with_series = site.posts | where_exp: "post", "post.series" %}
{% assign series_groups = posts_with_series | group_by: "series" %}

{% for series in series_groups %}
## {{ series.name }}

{% assign series_posts = series.items | sort: "part" %}
<ul>
  {% for post in series_posts %}
    <li>
      <a href="{{ post.url | relative_url }}">Part {{ post.part }}: {{ post.title }}</a>
      <span style="color:#666; font-size:0.9rem;">— {{ post.date | date: "%b %d, %Y" }}</span>
    </li>
  {% endfor %}
</ul>
{% endfor %}

{% assign posts_without_series = site.posts | where_exp: "post", "post.series == nil" %}
{% if posts_without_series.size > 0 %}
## Other Posts

<ul>
  {% for post in posts_without_series %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <span style="color:#666; font-size:0.9rem;">— {{ post.date | date: "%b %d, %Y" }}</span>
    </li>
  {% endfor %}
</ul>
{% endif %}
