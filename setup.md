---
layout: page
title: Data Science Development Setup
permalink: /setup/
weight: 5
sitemap:
  priority: 0.9
category: setup
---

<div class="page">

{% include page/title.html title=page.title subtitle=page_subtitle %}

    {% for post in site.posts %}

    {% assign category = site.my_categories | where: "slug", post.category %}
    {% assign category = category[0] %}
      {% if category %}
        {% capture category_content %}<a class="label" href="{{ category.url }}">{{ category.name }}</a>{% endcapture %}
      {% endif %}

      {% include page/post-list-item.html %}

      {% comment %}
      tagi w archiwum <span class="post-tag right">{{ tags_content }}</span>
      {% endcomment %}

     {% endfor %}
  	 </ul>


  {% comment %}
    {% include utils/tag-box.html %}
  {% endcomment %}


</div>
