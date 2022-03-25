---
layout: page
title: Blog
subtitle: My Digital Notebook
permalink: blog/
---

<ul>
  {% for post in collections.posts.resources %}
    <li>
      <a class="text-rose-700 underline decoration-dotted decoration-1 hover:no-underline" href="{{ post.relative_url }}">{{ post.data.title }}</a>
    </li>
  {% endfor %}
</ul>

