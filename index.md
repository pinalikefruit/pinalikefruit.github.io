---
layout: default
---
## DeFi Attack Vectors - Top 10 by Risk

This is a breakdown of the top DeFi attack vectors by funds lost. Select a year to view the corresponding data.

<!-- Tab Navigation Buttons -->
<ul class="tab-nav">
  <li class="active" data-tab="2024">2024 (Jan-Nov)</li>
  <li data-tab="2025">2025 (Jan-Nov)</li>
</ul>

<!-- Tab Content Panels -->
<div class="tab-content active" id="tab-2024">
  {% assign vectors_2024 = site.attack_vectors | where: 'year', 2024 | sort: 'rank' %}
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
      {% for vector in vectors_2024 %}
        <tr>
          <td class="rank">{{ vector.rank }}</td>
          <td class="name risk-{{ vector.risk_level }}"><a href="{{ vector.url | relative_url }}">{{ vector.title }}</a></td>
          <td class="amount">{{ vector.amount_stolen }}</td>
          <td class="incidents">{{ vector.incidents }}</td>
        </tr>
      {% endfor %}
    </tbody>
  </table>
</div>

<div class="tab-content" id="tab-2025">
  {% assign vectors_2025 = site.attack_vectors | where: 'year', 2025 | sort: 'rank' %}
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
      {% for vector in vectors_2025 %}
        <tr>
          <td class="rank">{{ vector.rank }}</td>
          <td class="name risk-{{ vector.risk_level }}"><a href="{{ vector.url | relative_url }}">{{ vector.title }}</a></td>
          <td class="amount">{{ vector.amount_stolen }}</td>
          <td class="incidents">{{ vector.incidents }}</td>
        </tr>
      {% endfor %}
    </tbody>
  </table>
</div>