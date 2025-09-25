---
layout: default
---
## Top 10 Attack Vectors

Here is a list of the top 10 attack vectors and the estimated amounts stolen. Click on each one to learn more about how to stay safe.

<ul>
  {% for vector in site.attack_vectors %}
    <li>
      <h3><a href="{{ vector.url | relative_url }}">{{ vector.title }}</a></h3>
      <p><strong>Amount Stolen:</strong> {{ vector.amount_stolen }}</p>
    </li>
  {% endfor %}
</ul>