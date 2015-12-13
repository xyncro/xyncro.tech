{% if include.project != null %}
{% assign updates = site.posts | where: "project", include.project %}
{% else %}
{% assign updates = site.posts %}
{% endif %}
{% for update in updates %}
{% assign project = site.data.projects[update.project] %}
<h2>{{ update.title }}</h2>
<dl>
  <dt>Project</dt>
  <dd>{{ project.name }}</dd>
  <dt>Date</dt>
  <dd>{{ update.date | date: "%d.%m.%Y" }}</dd>
</dl>
{{ update.content }}
{% endfor %}
