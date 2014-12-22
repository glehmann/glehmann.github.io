---
layout: page
title: Archive
---

{% for post in site.posts %}
  {% capture year %}{{ post.date | date: '%Y' }}{% endcapture %}
  {% if year != previousYear %}
## {{ year }}
  {% endif %}
  * {{ post.date | date_to_string }} &raquo; [ {{ post.title }} ]({{ post.url }})
  {% capture previousYear %}{{year}}{% endcapture %}
{% endfor %}
