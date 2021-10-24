<ul>
  {% for post in site.posts %}
    <article>
      <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
      {% if post.feature_img != "" %}
      <img src="{{ post.feature_img }}" />
      {% endif %}

    </article>
  {% endfor %}
</ul>
