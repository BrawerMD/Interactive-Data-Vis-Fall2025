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
    legend: true,
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

    // Honeybee SVG overlay
    Plot.image(
      closest.filter(d => d.pollinator_species === "Honeybee"),
      {
        x: "avg_body_mass_g",
        y: "avg_wing_span_mm",
        src: FileAttachment("data/honey.svg").url(),
        width: 40,
        height: 40
      }
    ),

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
      paintOrder: "stroke" // ensures the black outline is behind the white fill
    }),

    Plot.frame()
  ]
})
```





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
```js
Plot.plot({
  grid: true,
  x: {label: "Flower Species"},
  y: {label: "Average Nectar Production (μL)"},
  marks: [
    Plot.barY(
      Plot.groupY({y: "mean"}, data, {
        x: "flower_species",
        y: "nectar_production",
        fill: "#ff6900"
      })
    ),
    Plot.text(
      Plot.groupY({text: "mean"}, data, {
        x: "flower_species",
        y: "nectar_production",
        dy: -5,
        textAnchor: "middle",
        text: d => d.y.toFixed(2)
      })
    )
  ]
})
```
```js
import * as math from "npm:mathjs";

// Convert categorical weather_condition to dummy variables
const X = [];
const y = [];

const weatherOptions = [...new Set(data.map(d => d.weather_condition))]; // e.g., ["Sunny","Partly Cloudy","Cloudy"]

for (const d of data) {
  if (
    isFinite(d.nectar_production) &&
    isFinite(d.observation_hour) &&
    isFinite(d.temperature) &&
    isFinite(d.humidity) &&
    isFinite(d.wind_speed)
  ) {
    const row = [
      1, // intercept
      d.observation_hour,
      d.temperature,
      d.humidity,
      d.wind_speed,
      ...(weatherOptions.map(w => (d.weather_condition === w ? 1 : 0)))
    ];
    X.push(row);
    y.push(d.nectar_production);
  }
}

// Solve (X'X)^-1 X'y
const XT = math.transpose(X);
const XTX = math.multiply(XT, X);
const XTX_inv = math.inv(XTX);
const XTy = math.multiply(XT, y);
const coeffs = math.multiply(XTX_inv, XTy);

const headers = ["Intercept", "Hour", "Temp", "Humidity", "Wind", ...weatherOptions];
Plot.table({columns: ["Variable", "Coefficient"], rows: headers.map((v, i) => ({Variable: v, Coefficient: coeffs[i].toFixed(4)}))})
```
