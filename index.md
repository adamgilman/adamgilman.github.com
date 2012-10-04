---
layout: base
title: welcome to this

---
<h3>recent content</h3>
<p>
<ul>
  {% for post in site.posts reversed limit:10 %}
      <li>
          {{ post.date | date: "%B %Y" }} - [{{post.categories}}] - <a href="{{ post.url }}">{{ post.title }}</a>
      </li>
  {% endfor %}
</ul>
</p>  