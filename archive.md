---
layout: page
title: Archive
---

{% for post in site.posts %}	
   [{{post.title}}](<{{site.url}}{{ post.url}}>)
  <span class="post-date-small">{{ post.date | date_to_string }}</span>
{% endfor %} 