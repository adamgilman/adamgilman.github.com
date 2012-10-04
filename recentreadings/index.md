---
layout: base
title: Recent Readings
---
<h3>Collection of my recent readings</h3>

<p>This page really has no real function other than a final collection point of recent things that I've read that have caught my eye. It could be programming related, politics, economics, business, or just a great piece of journalism. There's no purpose or true intent for this collection other than my own personal archive of content for my own reference. If you find it interesting, than all the better for everyone.</p>

<p>
<ul>
	{% for post in site.categories.recentreadings %}
	    <li><a href="{{post.url}}">{{post.title}}</a></li>
	{% endfor %}
</ul>
</p>