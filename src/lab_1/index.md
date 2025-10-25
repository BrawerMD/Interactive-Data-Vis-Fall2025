---
title: "Lab 1: Passing Pollinators"
toc: true
---

# Studying Bee Species to Find the Perfect Pollination
<br>

### First, let's look at our 3 bee species
<br>

We observed...
```js
const beeColors = {
  Honeybee: "#f4b400",      // warm yellow
  Bumblebee: "#ff6900",     // orange
  "Carpenter Bee": "#3366cc" // deep blue
};
```


```js
const stats = data.reduce((acc, d) => {
  const s = d.pollinator_species;
  if (!acc[s]) acc[s] = { total_mass: 0, total_wing: 0, count: 0 };
  acc[s].total_mass += d.avg_body_mass_g;
  acc[s].total_wing += d.avg_wing_span_mm;
  acc[s].count++;
  return acc;
}, {});

for (const s in stats) {
  const v = stats[s];
  stats[s] = {
    avg_mass: (v.total_mass / v.count).toFixed(2),
    avg_wing: (v.total_wing / v.count).toFixed(1)
  };
}

stats
```
```html
<ul>
    <li><span style="color:${beeColors.Honeybee}; font-weight:600">
    Honeybees</span> — Smallest of the three species, averaging ${stats["Honeybee"].avg_mass}g body mass and ${stats["Honeybee"].avg_wing}mm avg. wingspan.
    <li><span style="color:${beeColors.Bumblebee}; font-weight:600">
    Bumblebees</span> — Larger, averaging ${stats["Bumblebee"].avg_mass}g body mass and ${stats["Bumblebee"].avg_wing}mm avg. wingspan.
    <li><span style="color:${beeColors["Carpenter Bee"]}; font-weight:600">
    Carpenters</span> — Largest, averaging ${stats["Carpenter Bee"].avg_mass}g body mass and ${stats["Carpenter Bee"].avg_wing}mm avg. wingspan.
</ul>
```

```js
const data = FileAttachment("data/pollinator_activity_data.csv").csv({typed: true});
```

```js
// Compute the averages first
const averages = data.reduce((acc, d) => {
  const s = d.pollinator_species;
  if (!acc[s]) acc[s] = { total_mass: 0, total_wing: 0, count: 0 };
  acc[s].total_mass += d.avg_body_mass_g;
  acc[s].total_wing += d.avg_wing_span_mm;
  acc[s].count++;
  return acc;
}, {});
for (const s in averages) {
  averages[s].avg_mass = averages[s].total_mass / averages[s].count;
  averages[s].avg_wing = averages[s].total_wing / averages[s].count;
}

// Find the one closest to each species' average
const closest = Object.values(
  data.reduce((acc, d) => {
    const s = d.pollinator_species;
    const avg = averages[s];
    const dist = Math.hypot(d.avg_body_mass_g - avg.avg_mass, d.avg_wing_span_mm - avg.avg_wing);
    if (!acc[s] || dist < acc[s].dist) acc[s] = {...d, dist};
    return acc;
  }, {})
);
closest
```

```js
Plot.plot({
  grid: true,
  inset: 10,
  title: "Wing Span & Body Mass (with Average Specimen per Bee Species)",
  x: { label: "Body Mass (g)" },
  y: { label: "Wing Span (mm)" },
  color: {
    legend: false,
    label: "Pollinator Species",
    domain: ["Honeybee", "Bumblebee", "Carpenter Bee"],
    range: ["#f4b400", "#ff6900", "#3366cc"]
  },
  marks: [
    // All dots
    Plot.dot(data, {
      x: "avg_body_mass_g",
      y: "avg_wing_span_mm",
      fill: "pollinator_species",
      r: 4,
      opacity: 0.5
    }),

    // Highlighted "closest to average" dots
    Plot.dot(closest, {
      x: "avg_body_mass_g",
      y: "avg_wing_span_mm",
      fill: "pollinator_species",
      r: 10,
      stroke: "black",
      strokeWidth: 1.5
    }),
    // 🏷️ Styled two-line labels
    Plot.text(closest, {
      x: "avg_body_mass_g",
      y: "avg_wing_span_mm",
      dy: -18,
      text: d =>
        `${d.pollinator_species}\n${d.avg_wing_span_mm.toFixed(1)} mm, ${d.avg_body_mass_g.toFixed(2)} g`,
      textAnchor: "middle",
      fontWeight: "bold",
      fill: "white",
      stroke: "black",
      strokeWidth: 4,
      paintOrder: "stroke"
    }),

    Plot.frame()
  ]
})
```
### Next we look at weather conditions
<br>
Starting with a regression of conditions & temperature on pollinator visits, we see that higher temperatures (but not maximally high) and clouds corrleate with visit count:


```js
Plot.plot({
  grid: true,
  color: {legend: true, label: "Weather Condition"},
  marks: [
    Plot.dot(data, {
      x: "temperature",
      y: "visit_count",
      fill: "weather_condition",
      r: 3,
      title: d => `${d.weather_condition}, ${d.temperature}°C – ${d.visit_count} visits`
    }),
    Plot.linearRegressionY(data, {
      x: "temperature",
      y: "visit_count",
      stroke: "weather_condition",
      strokeWidth: 2
    })
  ],
  x: {label: "Temperature (°C)"},
  y: {label: "Pollinator Visits"}
})
```
### Let's go deeper:

Average Visits by Temp:
```js
// 🌡️ Average Visit Count by Temperature (1°C bins)
Plot.plot({
  width: 800,
  height: 180,
  y: {grid: true, label: "Average Visit Count"},
  x: {label: "Temperature (°C)"},
  marks: [
    Plot.rectY(
      data,
      Plot.binX(
        {y: "mean"},
        {x: "temperature", y: "visit_count", fill: "#ffb000"}
      )
    ),
    Plot.ruleY([0])
  ]
})
```
Humidity:
```js
// 💧 Average Visit Count by Humidity (1% bins)
Plot.plot({
  width: 800,
  height: 180,
  y: {grid: true, label: "Average Visit Count"},
  x: {label: "Humidity (%)"},
  marks: [
    Plot.rectY(
      data,
      Plot.binX(
        {y: "mean"},
        {x: "humidity", y: "visit_count", fill: "#57bb8a"}
      )
    ),
    Plot.ruleY([0])
  ]
})
```
Wind Speed:
```js
// 🌬️ Average Visit Count by Wind Speed (1 km/h bins)
Plot.plot({
  width: 800,
  height: 180,
  y: {grid: true, label: "Average Visit Count"},
  x: {label: "Wind Speed (km/h)"},
  marks: [
    Plot.rectY(
      data,
      Plot.binX(
        {y: "mean"},
        {x: "wind_speed", y: "visit_count", fill: "#4e79a7"}
      )
    ),
    Plot.ruleY([0])
  ]
})
```
Hour:
```js
// 🕐 Average Visit Count by Observation Hour
Plot.plot({
  width: 800,
  height: 180,
  y: {grid: true, label: "Average Visit Count"},
  x: {label: "Observation Hour"},
  marks: [
    Plot.rectY(
      data,
      Plot.binX(
        {y: "mean"},
        {x: "observation_hour", y: "visit_count", fill: "#e15759"}
      )
    ),
    Plot.ruleY([0])
  ]
})
```
Taken together, we can reason that the best conditions for pollinating are **hot, cloudy, moderately humid (65-75% or so), and a breeze less than 4mph. These conditions are commonly found in 12p to 4p, our top hours of the day for pollination.**