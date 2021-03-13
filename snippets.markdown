---
layout: page
title: Snippets
permalink: /snippets/
---

# Snippets

Snippets page

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

aaa

<ul>
  {% for snippet in site.snippets %}
    <li>
      <a href="{{ snippet.url }}">{{ snippet.title }}</a>
    </li>
  {% endfor %}
</ul>