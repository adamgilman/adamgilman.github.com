---
layout: base
title: welcome to this
---
<h3>recent content</h3>
<p>
<ul>
  {% for post in site.posts limit:10 %}
      <li>
         <a href="{{ post.url }}">{{ post.title }}</a>
         {% if post.category != "blog" %}       	
			<blockquote>
			{{post.pullquote}}
			</blockquote>
         {% endif %}
      </li>
  {% endfor %}
</ul>
</p>  