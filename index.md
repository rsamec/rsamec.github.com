---
layout: page
title: state of mind!
tagline: design, develop 
---
{% include JB/setup %}
## About me

+ [LinkedIn](https://www.linkedin.com/pub/roman-samec/5b/372/423)
   
## Posts

How can this be a serious programming blog?
Can I actually learn this way?

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>



