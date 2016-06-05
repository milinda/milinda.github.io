---
layout: page
title: Blog
permalink: /blog/
---
<section class="sec-blogposts">
  <div class="container">
    <div class="row">
			<ul>
        {% for post in site.posts %}
        	{% if post.blog == true %}
          <li>
            <span class="date">{{ post.date | date: '%Y %b %d' }}</span> - <a class="post-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
          </li>
          {% endif %}
        {% endfor %}
      </ul>
    </div>
  </div>
</section>
