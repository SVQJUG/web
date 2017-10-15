---
layout: page
title: Posts
permalink: /posts/
---
<div class="container">
  <header class="post-header">
    <h1 class="post-title omeya_green_color" itemprop="name headline">Posts</h1>  
  </header>
  <ul class="collection">
    {% for post in site.posts %}
      <li class="collection-item">
        {% assign date_format = site.minima.date_format | default: "%d/%m" %}
          <div class="date-post">{{ post.date | date: date_format }}</div>
          <span class="title">
            <a class="post-link" href="{{ post.url | relative_url }}">{{ post.title | escape }}</a>
          </span>
          <a href="{{ post.url | relative_url }}" class="secondary-content"><i class="material-icons">navigate_next</i></a>
      </li>
    {% endfor %}
  </ul>
</div>
