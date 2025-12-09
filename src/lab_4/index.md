---
title: "Lab 4: Clearwater Crisis"
toc: false
---


```js
const wq = await FileAttachment("data/water_quality.csv").csv({typed: true});
wq.forEach(d => d.date = new Date(d.date));
const stations = d3.group(wq, d => d.station_name);

const suspectActivities = await FileAttachment("data/suspect_activities.csv").csv({typed: true});
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
    .style("fill", "white")  
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


svg.append("text")
  .attr("x", width/2)
  .attr("y", margin.top/2)
  .attr("text-anchor", "middle")
  .style("font-size", "20px")
  .style("fill", "white")
  .style("font-weight", "600")
  .text("Trout – Average Weight Over Time (by Station)");


const x = d3.scaleUtc()
  .domain(d3.extent(trout, d => d.date))
  .range([margin.left, width - margin.right]);

const y = d3.scaleLinear()
  .domain([400, 500])
  .range([height - margin.bottom, margin.top]);



svg.append("g")
  .attr("transform", `translate(0,${height - margin.bottom})`)
  .call(d3.axisBottom(x));

svg.append("g")
  .attr("transform", `translate(${margin.left},0)`)
  .call(d3.axisLeft(y));


for (const [station, values] of troutByStation) {
  values.sort((a,b) => a.date - b.date);


  svg.append("path")
    .datum(values)
    .attr("fill", "none")
    .attr("stroke", color(station))
    .attr("stroke-width", 2)
    .attr("d", d3.line()
      .x(d => x(d.date))
      .y(d => y(d.avg_weight_g))
    );

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

for (const [station, values] of troutByStation) {
  values.sort((a,b) => a.date - b.date);


  svg.append("path")
    .datum(values)
    .attr("fill", "none")
    .attr("stroke", color(station))
    .attr("stroke-width", 2)
    .attr("d", d3.line()
      .x(d => x(d.date))
      .y(d => y(d.count))
    );

  
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

Well, I think we should focus on what the West station is finding that kill the trout. The two steepest declines were also right after major spikes in Heavy Metals. We show how the % change in trout count moves period after priod to see the impact:

```js
const westWQ = wq.filter(d => d.station_id === "West");
const westTrout = fish
  .filter(d => d.species === "Trout" && d.station_id === "West");

westTrout.sort((a, b) => a.date - b.date);


for (let i = 1; i < westTrout.length; i++) {
  const prev = westTrout[i - 1].count;
  const curr = westTrout[i].count;
  westTrout[i].pctChange = ((curr - prev) / prev) * 100;
}

westTrout[0].pctChange = null;

```

```js
const width = 900;
const height = 400;
const margin = {top: 40, right: 80, bottom: 40, left: 60};

const svg = d3.create("svg")
  .attr("width", width)
  .attr("height", height);


svg.append("text")
  .attr("x", width / 2)
  .attr("y", margin.top / 2)
  .attr("text-anchor", "middle")
  .style("font-size", "20px")
  .style("font-weight", "600")
  .style("fill", "white")
  .text("West Station: Heavy Metals vs Trout Count Over Time");


const x = d3.scaleUtc()
  .domain(d3.extent(westWQ, d => d.date)) 
  .range([margin.left, width - margin.right]);


const yLeft = d3.scaleLinear()              
  .domain([0, d3.max(westWQ, d => d.heavy_metals_ppb)]).nice()
  .range([height - margin.bottom, margin.top]);

const yRight = d3.scaleLinear()           
  .domain([0, d3.max(westTrout, d => d.count)]).nice()
  .range([height - margin.bottom, margin.top]);


svg.append("g")
  .attr("transform", `translate(0,${height - margin.bottom})`)
  .call(d3.axisBottom(x))
  .selectAll("text")
  .style("fill", "white");

svg.append("g")
  .attr("transform", `translate(${margin.left},0)`)
  .call(d3.axisLeft(yLeft))
  .selectAll("text")
  .style("fill", "white");

svg.append("g")
  .attr("transform", `translate(${width - margin.right},0)`)
  .call(d3.axisRight(yRight))
  .selectAll("text")
  .style("fill", "white");

svg.append("g")
  .selectAll("text.pct")
  .data(westTrout)
  .enter()
  .append("text")
    .attr("class", "pct")
    .attr("x", d => x(d.date))
    .attr("y", d => yRight(d.count) - 10)
    .attr("text-anchor", "below")
    .style("font-size", "12px")
    .style("fill", "white")
    .text(d => {
      if (d.pctChange == null) return "";
      const val = d.pctChange.toFixed(1);
      return (val > 0 ? "+" : "") + val + "%";
    });



svg.append("path")
  .datum(westWQ)
  .attr("fill", "none")
  .attr("stroke", "red")
  .attr("stroke-width", 2)
  .attr("d", d3.line()
    .x(d => x(d.date))
    .y(d => yLeft(d.heavy_metals_ppb))
  );


svg.append("path")
  .datum(westTrout)
  .attr("fill", "none")
  .attr("stroke", "steelblue")
  .attr("stroke-width", 2)
  .attr("d", d3.line()
    .x(d => x(d.date))
    .y(d => yRight(d.count))
  );


const legend = svg.append("g")
  .attr("transform", `translate(${margin.left}, ${margin.top})`);

const legendItems = [
  {label: "Heavy Metals (ppb)", color: "red"},
  {label: "Trout Count", color: "steelblue"}
];

legendItems.forEach((d, i) => {
  const g = legend.append("g").attr("transform", `translate(0, ${i * 20})`);

  g.append("rect")
    .attr("width", 12)
    .attr("height", 12)
    .attr("fill", d.color);

  g.append("text")
    .attr("x", 18)
    .attr("y", 10)
    .style("font-size", "12px")
    .style("fill", "white")
    .text(d.label);
});

display(svg.node());
```

The West is plagued by ChemTech Manufacturing (Western Shore): time to figure out what they are up to.

```js

const chemtech = suspectActivities
  .filter(d => d.suspect === "ChemTech Manufacturing")
  .map(d => ({
    ...d,
    date: new Date(d.date),
    intensity_numeric:
      d.intensity === "Low" ? 1 :
      d.intensity === "Medium" ? 2 :
      3
  }));

```


```js
Inputs.table(
  chemtech.map(d => ({
    date: d.date,
    activity_type: d.activity_type,
    intensity: d.intensity,
    duration_days: d.duration_days,
    notes: d.notes
  }))
)
```
```js
const shutdowns = [
  {date:"2023-03-15", activity_type:"Maintenance Shutdown", intensity:"High", duration_days:7, notes:"Quarterly equipment maintenance and cleaning"},
  {date:"2023-06-20", activity_type:"Maintenance Shutdown", intensity:"High", duration_days:8, notes:"Quarterly equipment maintenance and cleaning"},
  {date:"2023-09-18", activity_type:"Maintenance Shutdown", intensity:"High", duration_days:7, notes:"Quarterly equipment maintenance and cleaning"},
  {date:"2023-12-10", activity_type:"Maintenance Shutdown", intensity:"High", duration_days:6, notes:"Year-end maintenance shutdown"},
  {date:"2024-03-12", activity_type:"Maintenance Shutdown", intensity:"High", duration_days:7, notes:"Quarterly equipment maintenance and cleaning"},
  {date:"2024-06-25", activity_type:"Maintenance Shutdown", intensity:"High", duration_days:9, notes:"Extended maintenance period"},
  {date:"2024-09-15", activity_type:"Maintenance Shutdown", intensity:"High", duration_days:7, notes:"Quarterly maintenance shutdown"},
  {date:"2024-12-08", activity_type:"Maintenance Shutdown", intensity:"High", duration_days:8, notes:"Year-end maintenance period"},
].map(d => ({
  ...d,
  start: new Date(d.date),
  end: new Date(new Date(d.date).getTime() + d.duration_days * 24*60*60*1000)
}));
```

If we get messy and put it all together:

I notice that yes, these shutdowns align perfectly with the peak in Heavy Metals.

AND: as research would tell us, the two species most susceptible to HMs -- Bass and Trout (but not Carp) -- are then in decline:

```JS

const westFish = fish
  .filter(d => d.station_id === "West")
  .map(d => ({...d, date: new Date(d.date)}));

const speciesGroups = d3.group(westFish, d => d.species);


const speciesColor = d3.scaleOrdinal()
  .domain(["Trout", "Bass", "Carp"])
  .range(["steelblue", "orange", "limegreen"]);

const shutdowns = [
  {date:"2023-03-15", activity_type:"Maintenance Shutdown", intensity:"High", duration_days:7, notes:"Quarterly equipment maintenance and cleaning"},
  {date:"2023-06-20", activity_type:"Maintenance Shutdown", intensity:"High", duration_days:8, notes:"Quarterly equipment maintenance and cleaning"},
  {date:"2023-09-18", activity_type:"Maintenance Shutdown", intensity:"High", duration_days:7, notes:"Quarterly equipment maintenance and cleaning"},
  {date:"2023-12-10", activity_type:"Maintenance Shutdown", intensity:"High", duration_days:6, notes:"Year-end maintenance shutdown"},
  {date:"2024-03-12", activity_type:"Maintenance Shutdown", intensity:"High", duration_days:7, notes:"Quarterly equipment maintenance and cleaning"},
  {date:"2024-06-25", activity_type:"Maintenance Shutdown", intensity:"High", duration_days:9, notes:"Extended maintenance period"},
  {date:"2024-09-15", activity_type:"Maintenance Shutdown", intensity:"High", duration_days:7, notes:"Quarterly maintenance shutdown"},
  {date:"2024-12-08", activity_type:"Maintenance Shutdown", intensity:"High", duration_days:8, notes:"Year-end maintenance period"},
].map(d => ({
  ...d,
  start: new Date(d.date)
}));

const width = 900;
const height = 450;
const margin = {top: 50, right: 80, bottom: 40, left: 60};

const svg = d3.create("svg")
  .attr("width", width)
  .attr("height", height);

svg.append("text")
  .attr("x", width/2)
  .attr("y", margin.top/2)
  .attr("text-anchor", "middle")
  .style("font-size","20px")
  .style("font-weight","600")
  .style("fill","white")
  .text("West Station: Fish Count Over Time (All Species)");

const x = d3.scaleUtc()
  .domain(d3.extent(westFish, d => d.date))
  .range([margin.left, width - margin.right]);

const y = d3.scaleLinear()
  .domain([0, d3.max(westFish, d => d.count)]).nice()
  .range([height - margin.bottom, margin.top]);

svg.append("g")
  .attr("transform", `translate(0,${height - margin.bottom})`)
  .call(d3.axisBottom(x))
  .selectAll("text")
  .style("fill","white");

svg.append("g")
  .attr("transform", `translate(${margin.left},0)`)
  .call(d3.axisLeft(y))
  .selectAll("text")
  .style("fill","white");

for (const [species, values] of speciesGroups) {
  values.sort((a,b) => a.date - b.date);

  svg.append("path")
    .datum(values)
    .attr("fill","none")
    .attr("stroke",speciesColor(species))
    .attr("stroke-width",2)
    .attr("d", d3.line()
      .x(d => x(d.date))
      .y(d => y(d.count))
    );

  svg.append("g")
    .selectAll("circle")
    .data(values)
    .enter()
    .append("circle")
      .attr("cx", d => x(d.date))
      .attr("cy", d => y(d.count))
      .attr("r",5)
      .attr("fill",speciesColor(species));
}

svg.append("g")
  .selectAll("line.shutdown")
  .data(shutdowns)
  .enter()
  .append("line")
    .attr("class","shutdown")
    .attr("x1", d => x(d.start))
    .attr("x2", d => x(d.start))
    .attr("y1", margin.top)
    .attr("y2", height - margin.bottom)
    .attr("stroke", "yellow")
    .attr("stroke-width", 2)
    .attr("stroke-dasharray", "4,2")
    .append("title")
      .text(d =>
        `${d.activity_type}
Intensity: ${d.intensity}
Duration: ${d.duration_days} days
${d.notes}`
      );

svg.append("g")
  .selectAll("text.shutdown-label")
  .data(shutdowns)
  .enter()
  .append("text")
    .attr("class","shutdown-label")
    .attr("x", d => x(d.start))
    .attr("y", margin.top - 5)
    .attr("text-anchor","middle")
    .style("font-size","11px")
    .style("fill","yellow")
    .text("Shutdown");

svg.append("path")
  .datum(westWQ)
  .attr("fill", "none")
  .attr("stroke", "red")
  .attr("stroke-width", 2)
  .attr("d", d3.line()
    .x(d => x(d.date))
    .y(d => yLeft(d.heavy_metals_ppb))
  );
// -------------------- LEGEND --------------------
const legend = svg.append("g")
  .attr("transform", `translate(${margin.left}, ${margin.top})`);

[
  {label:"Trout", color:"steelblue"},
  {label:"Bass",  color:"orange"},
  {label:"Carp",  color:"limegreen"},
  {label:"Maintenance Shutdown", color:"yellow"},
  {label:"Heavy Metals", color:"red"}
].forEach((item, i) => {
  const g = legend.append("g").attr("transform",`translate(0,${i*20})`);

  g.append("rect")
    .attr("width",12)
    .attr("height",12)
    .attr("fill",item.color);

  g.append("text")
    .attr("x",18)
    .attr("y",10)
    .style("font-size","12px")
    .style("fill","white")
    .text(item.label);
});

display(svg.node());

```

Mystery solved: the Clearwater Crisis may be caused in part by ChemTech's behaviors while doing "maintenance" (hmm). It seems they shut down just as their heavy metals are peaking (or allow them to peak as part of the shutdown protocol), then they let things die down before they go back to polluting.

Shame, shame, shame!