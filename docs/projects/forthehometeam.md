---
layout: page
permalink: /projects/forthehometeam
---

I am convinced that in my life time, I have seen my favorite teams lose more than win, when I actively attend or watch games. Rather than believe the likely explanation that my favorite teams simply underperform, I've chosen to spend many hours
tracking the Cubs games I watch and seeing how the numbers stack up throughout the 2023 season. This naturally spiraled into comparing not just wins and losses, but also the stats of individual players.

These stats are current as of June 19th, 2023. (And hopefully this will be automated in the near future.)

### Win Rate

<table>
  {% for row in site.data.record %}
    {% if forloop.first %}
    <tr>
      {% for pair in row %}
        <th>{{ pair[0] }}</th>
      {% endfor %}
    </tr>
    {% endif %}

    {% tablerow pair in row %}
      {{ pair[1] | round: 2 }}
    {% endtablerow %}
  {% endfor %}
</table>

### Batting Averages

Below are players who have a difference of at least one standard deviation from their seasonal batting average in the games I've watched.
Additionally, I've filtered down the list to only include players for whom I've seen a decent number of at bats. Through advanced statistical analysis
(Arbitrarily choosing a number that felt right) I've settled on only players whom I've seen at bat more than 10 times.

<table>
  {% for row in site.data.batting_avg %}
    {% if forloop.first %}
    <tr>
      {% for pair in row %}
        <th>{{ pair[0] }}</th>
      {% endfor %}
    </tr>
    {% endif %}

    {% tablerow pair in row %}
      {{ pair[1] }}
    {% endtablerow %}
  {% endfor %}
</table>
