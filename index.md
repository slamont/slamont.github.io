# Personal Notes

## Today I Learned

{% for post in site.posts %}
 * [{{ post.date | date_to_string }}]({{ post.url }}) {{ post.title }}
{% endfor %}
