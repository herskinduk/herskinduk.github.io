---
layout: homepage
title: The Aproppos Sitecore Blog
tagline: observations and reflections on the kernel    
---
{% include JB/setup %}

  {% for post in site.posts | limit: 10 %}
<article>
  <header>
      <h1><a href="{{ post.url | prepend: site.baseurl }}">{{post.title}}</a></h1>
      <div class="date">
        <span>Posted {% assign prettydate = post.date %}{% include JB/pretty_date %} by {{site.author.name}}</span>
      </div>
  </header>
{% if post.content contains '<!--more-->' %}
	{{ post.content | split:'<!--more-->' | first }}
{% else %}
	{{ post.excerpt }}
{% endif %}
  <p><a href="{{ post.url | prepend: site.baseurl }}">Read more...</a></p>
</article>
  {% endfor %}

<hr>
<a href="archive.html">Older posts</a>
