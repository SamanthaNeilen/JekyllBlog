---
layout: page
title: Archive
permalink: /archive/
---
<table class="table">	
	<tr>
		<th>Title</th>
		<th>Date</th>
		<th>Tags</th>
	</tr>
{% for post in site.posts  %}
	{% assign date_format = siteminima.date_format | default: "%b %-d, %Y" %}
    <tr>
		<td><a href="{{ post.url | relative_url }}">{{ post.title | escape }}</a></td>
		<td>{{ post.date | date: date_format }}</td>
		<td>
			{% for tag in post.tags %}		
				{{tag}} 
			{% endfor %}
		</td>
	</tr>	
{% endfor %}
</table>

