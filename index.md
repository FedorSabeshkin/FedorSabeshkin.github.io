<ul>
  {% for post in site.posts %}
    <li>
      <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
      {% if post.feature_img != "" %}
      <img src="{{ post.feature_img }}" />
      {% endif %}
      {{ post.content | strip_html | truncatewords: 50 }}
    </li>
  {% endfor %}
</ul>
