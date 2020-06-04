---
layout: default
---

Welcome to my brand new blog - created 2020-06-04

{% for post in site.posts %}
<div>
    {{ post.date  | date: "%Y-%m-%d" }} <a href="{{ post.url }}">{{ post.title }}</a>
</div>
    {% endfor %}
