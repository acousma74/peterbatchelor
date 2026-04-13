---
layout: default
title: Home
permalink: /index.html
---
<h1>news</h1>
<table class="news">
  {% for post in site.posts %}
  <tr class="news">
    <td class="news" width="100%">
      <h1 class="news">{{ post.title }}</h1>
      <h2 class="newsDate">{{ post.date | date: "%d.%m.%y" }}</h2>
      
      <div class="news-summary">
        {{ post.excerpt }}
      </div>
      
        {% if post.image %}
        <div class="news-image-window">
          <img src="{{ site.baseurl }}/assets/images/{{ post.image }}" alt="{{ post.title }}">
        </div>
        {% endif %}

        <div class="news-text-content">
          <div class="news-summary">
            {{ post.excerpt }}
          </div>
          <p><a href="{{ site.baseurl }}{{ post.url }}">Read more...</a></p>
        </div>

      <p><a href="{{ site.baseurl }}{{ post.url }}">Read more...</a></p>
    </td>
  </tr>
  {% endfor %}
</table>