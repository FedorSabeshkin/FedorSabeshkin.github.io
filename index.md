<ul>
  {% for post in site.posts %}
    <li>
      <img src="{{ post.feature_img }}" />
      <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
    </li>
  {% endfor %}
</ul>
