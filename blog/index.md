---
layout: base
title: Blog Posts
---
<h3>Blog Posts</h3>

<p>
<ul>
	{% for post in site.categories.blog %}
	    <li><a href="{{post.url}}">{{post.title}}</a></li>
	{% endfor %}
</ul>
</p>