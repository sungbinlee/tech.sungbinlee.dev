---
title: "Django"
layout: archive
permalink: /django
---


{% assign posts = site.categories['Django'] %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}