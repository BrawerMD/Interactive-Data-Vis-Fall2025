---
title: "Lab 4: Clearwater Crisis"
toc: false
---


```js
const wq = await FileAttachment("data/water_quality.csv").csv({typed: true});
wq.forEach(d => d.date = new Date(d.date));
const stations = d3.group(wq, d => d.station_name);

```

```js

const stationOrder = ["North", "South", "East", "West"];

const color = d3.scaleOrdinal()
  .domain(stationOrder)
  .range(d3.schemeTableau10.slice(0, stationOrder.length));


```
# Clearwater CRISIS
<br>
To begin our mystery, we state the problem: which station is a likely culprit for polluting our lake? The first question is to see what "polluted" means -- does looking at reads of pollutants over time by station reveal any hotspots?

<br><br>



### CONTAMINATES OVER TIME:
<br>

### Nitrogen
```js
//Chart 1 
const width = 900;
const height = 400;
const margin = {top: 30, right: 30, bottom: 40, left: 60};

const svg = d3.create("svg")
  .attr("width", width)
  .attr("height", height);

const x = d3.scaleUtc()
  .domain(d3.extent(wq, d => d.date))
  .range([margin.left, width - margin.right]);

const y = d3.scaleLinear()
  .domain([0, d3.max(wq, d => d.nitrogen_mg_per_L)]).nice()
  .range([height - margin.bottom, margin.top]);

svg.append("g")
  .attr("transform", `translate(0,${height - margin.bottom})`)
  .call(d3.axisBottom(x));

svg.append("g")
  .attr("transform", `translate(${margin.left},0)`)
  .call(d3.axisLeft(y));

// lines for each station
for (const [station, values] of stations) {
  svg.append("path")
    .datum(values)
    .attr("fill", "none")
    .attr("stroke", color(station))
    .attr("stroke-width", 2)
    .attr("d", d3.line()
      .x(d => x(d.date))
      .y(d => y(d.nitrogen_mg_per_L))
    );

  svg.append("text")
    .attr("x", x(values[values.length - 1].date) + 5)
    .attr("y", y(values[values.length - 1].nitrogen_mg_per_L))
    .attr("dy", "0.35em")
    .attr("fill", color(station))
    .style("font-size", "12px")
    .text(station);
}
    // ---- Legend ----
const legend = svg.append("g")
  .attr("transform", `translate(${margin.left + 10}, ${margin.top})`);

Array.from(stations.keys()).forEach((station, i) => {
  const g = legend.append("g")
    .attr("transform", `translate(0, ${i * 20})`);

  g.append("rect")
    .attr("width", 12)
    .attr("height", 12)
    .attr("fill", color(station));

  g.append("text")
    .attr("x", 18)
    .attr("y", 10)
    .style("font-size", "12px")
    .style("fill", "white")   // adjust to your dark background
    .text(station);
});



display(svg.node());
```
### Phosphorus
```js
//chart 2
const width = 900;
const height = 400;
const margin = {top: 30, right: 30, bottom: 40, left: 60};

const svg = d3.create("svg")
  .attr("width", width)
  .attr("height", height);

const x = d3.scaleUtc()
  .domain(d3.extent(wq, d => d.date))
  .range([margin.left, width - margin.right]);

const y = d3.scaleLinear()
  .domain([0, d3.max(wq, d => d.phosphorus_mg_per_L	)]).nice()
  .range([height - margin.bottom, margin.top]);

svg.append("g")
  .attr("transform", `translate(0,${height - margin.bottom})`)
  .call(d3.axisBottom(x));

svg.append("g")
  .attr("transform", `translate(${margin.left},0)`)
  .call(d3.axisLeft(y));

// lines for each station
for (const [station, values] of stations) {
  svg.append("path")
    .datum(values)
    .attr("fill", "none")
    .attr("stroke", color(station))
    .attr("stroke-width", 2)
    .attr("d", d3.line()
      .x(d => x(d.date))
      .y(d => y(d.phosphorus_mg_per_L	))
    );

  svg.append("text")
    .attr("x", x(values[values.length - 1].date) + 5)
    .attr("y", y(values[values.length - 1].phosphorus_mg_per_L))
    .attr("dy", "0.35em")
    .attr("fill", color(station))
    .style("font-size", "12px")
    .text(station);
}

display(svg.node());
```
### Heavy Metal
```js
//chart 3
const width = 900;
const height = 400;
const margin = {top: 30, right: 30, bottom: 40, left: 60};

const svg = d3.create("svg")
  .attr("width", width)
  .attr("height", height);

const x = d3.scaleUtc()
  .domain(d3.extent(wq, d => d.date))
  .range([margin.left, width - margin.right]);

const y = d3.scaleLinear()
  .domain([0, d3.max(wq, d => d.heavy_metals_ppb	)]).nice()
  .range([height - margin.bottom, margin.top]);

svg.append("g")
  .attr("transform", `translate(0,${height - margin.bottom})`)
  .call(d3.axisBottom(x));

svg.append("g")
  .attr("transform", `translate(${margin.left},0)`)
  .call(d3.axisLeft(y));

// lines for each station
for (const [station, values] of stations) {
  svg.append("path")
    .datum(values)
    .attr("fill", "none")
    .attr("stroke", color(station))
    .attr("stroke-width", 2)
    .attr("d", d3.line()
      .x(d => x(d.date))
      .y(d => y(d.heavy_metals_ppb
	))
    );

  svg.append("text")
    .attr("x", x(values[values.length - 1].date) + 5)
    .attr("y", y(values[values.length - 1].heavy_metals_ppb
))
    .attr("dy", "0.35em")
    .attr("fill", color(station))
    .style("font-size", "12px")
    .text(station);
}

display(svg.node());
```

Sure does. The North area of the lake is a constant offender, spiking in April & October (seasonal) with pollutants.

With that said, the West area ends up with major heavy metal spikes slighlty earlier, and the East gets a lot of phosphorus contaimation just after.

So which of these pollutants are the problem? We want to know specifically WHAT HAPPENED TO THE FISH! AND WHY?

Let's focus on Trout, my favorite:

```js
const fish = await FileAttachment("data/fish_surveys.csv").csv({typed:true});
fish.forEach(d => d.date = new Date(d.date));

const speciesGroups = d3.group(fish, d => d.species);
const speciesColor = d3.scaleOrdinal()
  .domain([...speciesGroups.keys()])
  .range(d3.schemeTableau10);

const trout = fish.filter(d => d.species === "Trout");
const troutByStation = d3.group(trout, d => d.station_id);

```
```js
const width = 900;
const height = 400;
const margin = {top: 40, right: 80, bottom: 40, left: 60};

const svg = d3.create("svg")
  .attr("width", width)
  .attr("height", height);

// Title
svg.append("text")
  .attr("x", width/2)
  .attr("y", margin.top/2)
  .attr("text-anchor", "middle")
  .style("font-size", "20px")
  .style("fill", "white")
  .style("font-weight", "600")
  .text("Trout – Average Weight Over Time (by Station)");

// Scales
const x = d3.scaleUtc()
  .domain(d3.extent(trout, d => d.date))
  .range([margin.left, width - margin.right]);

const y = d3.scaleLinear()
  .domain([400, 500])
  .range([height - margin.bottom, margin.top]);


// Axes
svg.append("g")
  .attr("transform", `translate(0,${height - margin.bottom})`)
  .call(d3.axisBottom(x));

svg.append("g")
  .attr("transform", `translate(${margin.left},0)`)
  .call(d3.axisLeft(y));

// Draw lines + points
for (const [station, values] of troutByStation) {
  values.sort((a,b) => a.date - b.date);

  // Line
  svg.append("path")
    .datum(values)
    .attr("fill", "none")
    .attr("stroke", color(station))
    .attr("stroke-width", 2)
    .attr("d", d3.line()
      .x(d => x(d.date))
      .y(d => y(d.avg_weight_g))
    );

  // Scatter points
  svg.append("g")
    .selectAll("circle")
    .data(values)
    .enter()
    .append("circle")
      .attr("cx", d => x(d.date))
      .attr("cy", d => y(d.avg_weight_g))
      .attr("r", 5)
      .attr("fill", color(station))
      .append("title")
        .text(d =>
          `${station}
${d3.timeFormat("%b %Y")(d.date)}
Avg Weight: ${d.avg_weight_g} g`
        );
}

// Legend (top-left)
const legend = svg.append("g")
  .attr("transform", `translate(${margin.left + 10}, ${margin.top})`);

["West","South","East","North"].forEach((station, i) => {
  const g = legend.append("g")
    .attr("transform", `translate(0, ${i * 20})`);

  g.append("rect")
    .attr("width", 12)
    .attr("height", 12)
    .attr("fill", color(station));

  g.append("text")
    .attr("x", 18)
    .attr("y", 10)
    .style("font-size", "12px")
    .style("fill", "white")
    .text(station);
});

display(svg.node());

```
_psst: we know the line runs off the min Y axis, it looks catastrophic that way_

```js
const width = 900;
const height = 400;
const margin = {top: 40, right: 80, bottom: 40, left: 60};

const svg = d3.create("svg")
  .attr("width", width)
  .attr("height", height);

// Title
svg.append("text")
  .attr("x", width/2)
  .attr("y", margin.top/2)
  .attr("text-anchor", "middle")
  .style("font-size", "20px")
  .style("fill", "white")
  .style("font-weight", "600")
  .text("Trout – Count Decline Over Time (by Station)");

const x = d3.scaleUtc()
  .domain(d3.extent(trout, d => d.date))
  .range([margin.left, width - margin.right]);

const y = d3.scaleLinear()
  .domain([0, d3.max(trout, d => d.count)]).nice()
  .range([height - margin.bottom, margin.top]);

svg.append("g")
  .attr("transform", `translate(0, ${height - margin.bottom})`)
  .call(d3.axisBottom(x));

svg.append("g")
  .attr("transform", `translate(${margin.left},0)`)
  .call(d3.axisLeft(y));

// Draw lines + scatter per station
for (const [station, values] of troutByStation) {
  values.sort((a,b) => a.date - b.date);

  // line
  svg.append("path")
    .datum(values)
    .attr("fill", "none")
    .attr("stroke", color(station))
    .attr("stroke-width", 2)
    .attr("d", d3.line()
      .x(d => x(d.date))
      .y(d => y(d.count))
    );

  // points
  svg.append("g")
    .selectAll("circle")
    .data(values)
    .enter()
    .append("circle")
      .attr("cx", d => x(d.date))
      .attr("cy", d => y(d.count))
      .attr("r", 5)
      .attr("fill", color(station))
      .append("title")
        .text(d =>
          `${station}
${d3.timeFormat("%b %Y")(d.date)}
Count: ${d.count}`
        );
}

// Legend
const legend = svg.append("g")
  .attr("transform", `translate(${margin.left + 10}, ${margin.top})`);

["West","South","East","North"].forEach((station, i) => {
  const g = legend.append("g")
    .attr("transform", `translate(0, ${i * 20})`);

  g.append("rect")
    .attr("width", 12)
    .attr("height", 12)
    .attr("fill", color(station));

  g.append("text")
    .attr("x", 18)
    .attr("y", 10)
    .style("font-size", "12px")
    .style("fill", "white")
    .text(station);
});

display(svg.node());

```

Well, I think we should focus on what the West station is finding that kill the trout.