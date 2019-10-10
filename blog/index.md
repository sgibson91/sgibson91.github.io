---
layout: category
title: Blog
search_omit: true
image:
  path: /images/blog.png
  caption: "Photo by [Patrick Tomasso](https://unsplash.com/@impatrickt?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/blog?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)"
---

<table style="width:100%">
  <!-- <tr>
    <th>Date</th>
    <th>Title</th>
    <th>Excerpt</th>
  </tr> -->
  {% for post in site.categories.blog %}
    <tr>
      <th>{{ post.date | date: "%B %d, %Y" }}</th>
      <th><a href="{{ post.url }}">{{ post.title }}</a></th>
      <th>{% if post.excerpt %} {{ post.excerpt | remove: '\[ ... \]' | remove: '\( ... \)' | markdownify | strip_html | strip_newlines | escape_once }} {% endif %}</th>
    </tr>
  {% endfor %}
</table>
