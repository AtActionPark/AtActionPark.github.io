---
layout: page
title: Blog
---

{% for post in site.posts %}
  * {{ post.date | date_to_string }} &raquo; [ {{ post.title }} ]({{ post.url }})
{% endfor %}

<div class="posts">
  {% for post in paginator.posts %}
  <article class="post">
    <h1 class="post-title">
      <a href="{{ site.baseurl }}{{ post.url }}">
        {{ post.title }}
      </a>
    </h1>

    <time datetime="{{ post.date | date_to_xmlschema }}" class="post-date">{{ post.date | date_to_string }}</time>

    <p class="post-excerpt">{{ post.excerpt | strip_html }}</p>
    <a href="{{ post.url }}"> Read more </a>
  </article>
  {% endfor %}
</div>

<div class="pagination">
  {% if paginator.next_page %}
    <a class="pagination-item older" href="{{ paginator.next_page_path | prepend: site.baseurl }}">Older</a>
  {% else %}
    <span class="pagination-item older">Older</span>
  {% endif %}
  {% if paginator.previous_page %}
    <a class="pagination-item newer" href="{{ paginator.previous_page_path | prepend: site.baseurl }}">Newer</a>
  {% else %}
    <span class="pagination-item newer">Newer</span>
  {% endif %}
</div>