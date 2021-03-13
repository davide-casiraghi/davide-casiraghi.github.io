---
layout: page
title: Snippets
permalink: /snippets/
---

# Snippets

Snippets are little pieces of code that are an example already tested to provide and example of use.


<ul>
  {% for snippet in site.snippets %}
    <li>
      <a href="{{ snippet.url }}">{{ snippet.title }}</a>
    </li>
  {% endfor %}
</ul>
