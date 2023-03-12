---
layout: archive
permalink: /categories/CS
title: "CS"
author_profile: true
---
{% include group-by-array collection=site.posts field="categories" %}
{% for category in group_names %}
  {% assign posts = group_items[forloop.index0] %}
  {% if category == "CS" %}

    {% for post in posts %}
      {% include archive-single.html %}
    {% endfor %}
  {% endif %}
{% endfor %}
