---
layout: default
---

{% for post in site.posts %}
<div>
    {{ post.date  | date: "%Y-%m-%d" }} <a href="{{ post.url }}">{{ post.title }}</a>
</div>
    {% endfor %}