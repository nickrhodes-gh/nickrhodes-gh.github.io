---
layout: default
title: Archived blog posts
---
<div class="page-content wc-container">
  <h3>Blog Archive</h3>
  {% for post in site.posts %}
  	{% capture currentyear %}{{post.date | date: "%Y"}}{% endcapture %}
  	{% if currentyear != year %}
    	{% unless forloop.first %}</ul>{% endunless %}
   		<h4>{{ currentyear }}</h4>
   		<ul class="posts">
   		{% capture year %}{{currentyear}}{% endcapture %}
	{% endif %}
    <li>- <a href="{{ site.url }}{{ post.url }}">{{ post.title }}</a></li>
    {% if forloop.last %}</ul>{% endif %}
{% endfor %}
</div>
