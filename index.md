<ul>
  {% for post in site.posts %}
    <article class="post">
      <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
      {% if post.feature_img != "" %}
      <img src="{{ post.feature_img }}" />
      {% endif %}
      <p>
            {{ post.content | markdownify | strip_html | truncatewords: 50 }}
      </p>
    </article>
  {% endfor %}
</ul>
