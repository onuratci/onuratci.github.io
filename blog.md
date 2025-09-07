---
layout: default
title: Blog
permalink: /blog/
---

# Blog

Welcome to my blog where I share insights on software development, engineering management, cloud technologies, and lessons learned from building scalable systems.

<div class="blog-posts">
{% for post in site.posts %}
  <article class="post-item">
    <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
    <p class="post-meta">
      <time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%B %d, %Y" }}</time>
      {% if post.categories.size > 0 %}
        • 
        {% for category in post.categories %}
          <span class="category">{{ category }}</span>{% unless forloop.last %}, {% endunless %}
        {% endfor %}
      {% endif %}
    </p>
    <p class="post-excerpt">{{ post.excerpt | strip_html | truncatewords: 50 }}</p>
    <p><a href="{{ post.url }}" class="read-more">Read more →</a></p>
  </article>
{% endfor %}
</div>

{% if site.posts.size == 0 %}
<div class="no-posts">
  <p>No posts yet! Check back soon for insights on software development, engineering management, and technology.</p>
</div>
{% endif %}