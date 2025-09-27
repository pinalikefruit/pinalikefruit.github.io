---
layout: default
---
## DeFi Attack Vectors (2024 Jan-November) - Top 10 by Risk

This is a breakdown of the top 10 DeFi attack vectors by funds lost. Click on any vector to read a detailed analysis and learn how to stay safe.

{% assign sorted_vectors = site.attack_vectors | sort: 'rank' %}

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
    {% for vector in sorted_vectors %}
      <tr>
        <td class="rank">{{ vector.rank }}</td>
        <td class="name {{ vector.risk_level }}"><a href="{{ vector.url | relative_url }}">{{ vector.title }}</a></td>
        <td class="amount">{{ vector.amount_stolen }}</td>
        <td class="incidents">{{ vector.incidents }}</td>
      </tr>
    {% endfor %}
  </tbody>
</table>