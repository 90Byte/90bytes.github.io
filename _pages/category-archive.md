---
title: "Featured Posts"
layout: single
permalink: /featured/
author_profile: true
---
{{- site.sidebar -}}
{%- assign fp = site.posts | where: 'featuredPost', true -%}
<div class="entries-{{ entries_layout }}">
      {% for post in fp %}
        {% include archive-single.html type=entries_layout %}
      {% endfor %}
</div>