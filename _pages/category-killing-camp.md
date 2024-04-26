---
title: "Algorithms"
layout: archive
permalink: /algorithms
---


{% assign posts = site.categories['Algorithms'] %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}
