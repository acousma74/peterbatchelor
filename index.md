---
layout: default
title: Home
permalink: /index.html
---

<table class="news">
  {% for post in site.posts %}
  <tr class="news">
    <td class="news" width="100%">
      <h1 class="news">{{ post.title }}</h1>
      <h2 class="newsDate">{{ post.date | date: "%d.%m.%y" }}</h2>
      
      {{ post.content }}
      
      {% if post.image %}
      <p align="center">
        <img src="{{ site.baseurl }}/assets/images/{{ post.image }}" alt="{{ post.title }}" />
      </p>
      {% endif %}
    </td>
  </tr>
  {% endfor %}
</table>