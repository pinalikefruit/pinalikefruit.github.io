---
layout: default
---

A constantly updated guide that compiles best practices from security experts for **developers** in one place, helping them avoid attacks.


<ul class="tab-nav">
  <li class="active" data-tab="2024">2024</li>
  <li data-tab="2025">2025</li>
</ul>


{% for year in (2024..2025) %}
  <div class="tab-content {% if year == 2024 %}active{% endif %}" id="tab-{{ year }}">
    {% assign ranked_items = site.data.rankings | where: 'year', year | sort: 'rank' %}
    <table class="vector-table">
      <thead>
        <tr>
          <th>#</th>
          <th>Attack Vector</th>
          <th style="text-align: right;">Amount Stolen</th>
          <th style="text-align: right;">Incidents</th>
        </tr>
      </thead>
      <tbody>
        {% for item in ranked_items %}
          {% assign vector_article = site.attack_vectors | where: "slug", item.id | first %}
          <tr>
            <td class="rank">{{ item.rank }}</td>
            <td class="name risk-{{ item.risk_level }}">
              {% if item.unlocked == true and vector_article %}
                <a href="{{ vector_article.url | relative_url }}">{{ vector_article.title }}</a>
              {% else %}
                <span>{{ vector_article.title | default: item.id | replace: "-", " " | capitalize }} (Coming Soon)</span>
              {% endif %}
            </td>
            <td class="amount">{{ item.amount_stolen }}</td>
            <td class="incidents">{{ item.incidents }}</td>
          </tr>
        {% endfor %}
      </tbody>
    </table>
  </div>
{% endfor %}