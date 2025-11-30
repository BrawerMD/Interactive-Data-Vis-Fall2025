---
title: "Lab 3: Mayoral Mystery"
toc: false
---

<!-- Import Data -->
```js
const nyc = await FileAttachment("data/nyc.json").json();
const results = await FileAttachment("data/election_results.csv").csv({ typed: true });
const survey = await FileAttachment("data/survey_responses.csv").csv({ typed: true });
const events = await FileAttachment("data/campaign_events.csv").csv({ typed: true });
const districts = topojson.feature(nyc, nyc.objects.districts);
```

# Mayoral Mystery

For this exercise, we imagined we were trying to consult with the candidate on a very focused topic -- how to use their time next year.

To answer this question of how best to use time, we focused our story on the MARGINS -- like any good data scientist, we want to know the marginal value of time spent in a district.

We use a regression to create an "effort score", or the value of the time the candidate spent in an district on earning marginal votes. We plot that score here from low (white) to high (blue) and see where effort paid off:

```js
//----------------------------------------------
// 0. Base District Stats (same pattern as working map)
//----------------------------------------------
const voteShare = results.map(d => ({
  boro_cd: d.boro_cd,                                  // KEY
  income: d.income_category,
  turnout: d.turnout_rate,
  share: d.votes_candidate / (d.votes_candidate + d.votes_opponent),
  median_income: d.median_household_income
}));

const districtStats = new Map(voteShare.map(d => [d.boro_cd, d]));


//----------------------------------------------
// 1. Effort Score (residual of votes ~ hours)
//----------------------------------------------
const effortData = results.map(d => ({
  boro_cd: d.boro_cd,
  hours: d.candidate_hours_spent,
  votes: d.votes_candidate
}));

// regression: votes ~ hours
const meanHours = d3.mean(effortData, d => d.hours);
const meanVotes = d3.mean(effortData, d => d.votes);

let cov = 0, varH = 0;
effortData.forEach(d => {
  cov += (d.hours - meanHours) * (d.votes - meanVotes);
  varH += (d.hours - meanHours) ** 2;
});

const slope_hours = cov / varH;
const intercept_hours = meanVotes - slope_hours * meanHours;

const efficiency = effortData.map(d => {
  const predicted = intercept_hours + slope_hours * d.hours;
  return {
    ...d,
    predicted,
    residual: d.votes - predicted     // >0 = over-performing district
  };
});

const efficiencyMap = new Map(efficiency.map(d => [d.boro_cd, d]));
const residualExtent = d3.extent(efficiency, d => d.residual);


//----------------------------------------------
// 2. Centroids (same pattern as working chart)
//----------------------------------------------
const districtCentroids = districts.features
  .map(feature => {
    const stats = districtStats.get(feature.properties.BoroCD);
    if (!stats) return null;
    return {
      ...stats,
      coordinates: d3.geoCentroid(feature)
    };
  })
  .filter(Boolean);


//----------------------------------------------
// 3. Income color palette for centroids
//----------------------------------------------
const incomeColors = {
  Low: "#d62828",      // red
  Middle: "#fcd34d",   // yellow
  High: "#16a34a"      // green
};

```

```js
const effortScoreMap = Plot.plot({
  title: "Effort Score: Impact of Candidate Time on Vote Performance",
  projection: {
    domain: districts,
    type: "mercator"
  },
  height: 720,
  width: 800,
  color: {
    type: "sequential",
    scheme: "Blues",            // white → blue
    domain: residualExtent,
    label: "Excess votes vs expected (residual)"
  },
  marks: [
    // ---- District polygons with tooltip ----
    Plot.geo(districts, {
      fill: d => {
        const key = d.properties.BoroCD;
        return efficiencyMap.get(key)?.residual;
      },
      stroke: "#f8f9fa",
      strokeWidth: 0.6,
      tip: true,
      title: d => {
        const key = d.properties.BoroCD;
        const base = districtStats.get(key);
        const eff  = efficiencyMap.get(key);

        if (!eff) return `District ${key}\nNo efficiency data`;

        return `District ${key}
Income tier: ${base.income}
Vote share: ${(base.share * 100).toFixed(1)}%
Votes: ${eff.votes}
Expected: ${eff.predicted.toFixed(0)}
Effort score: ${eff.residual.toFixed(0)}`;
      }
    }),

    // ---- Centroid dots (income colors, NO tooltip) ----
    Plot.dot(districtCentroids, {
      x: d => d.coordinates[0],
      y: d => d.coordinates[1],
      fill: d => incomeColors[d.income],
      r:4,
    }),
  ]
});

display(effortScoreMap);
```
```js
display(Plot.legend({
  color: {
    type: "sequential",
    scheme: "Blues",
    domain: residualExtent,
    label: "Excess votes vs expected (residual)"
  }
}));

```
```js

display(Plot.legend({
  color: {
    type: "ordinal",
    domain: ["Low Income", "Middle Income", "High Income"],
    range: ["#d62828", "#fcd34d", "#16a34a"],
    
  }
}));
```

As a New York-based firm, we quickly noticed something about the darker (more rewarding) districts. To confirm, we plotted their income category in dots, green through red. We observe an inverse correlation between wealth and marginal value of effort.

### Put simply, we recommend the candidate spend more time in these lower-income areas, followed by middle-income.
<br>
---

To follow up on this momentum in said districts, we looked to see how specific event types performed. Binning by type, we see the average attendance of event type in low-income areas:

```js
//----------------------------------------------
// 1. Filter to ONLY low-income events
//----------------------------------------------
const lowIncomeEvents = events
  .filter(e => e.income_category === "Low")
  .sort((a, b) => b.estimated_attendance - a.estimated_attendance);  // descending


//----------------------------------------------
// 2. Create color mapping for event type
//----------------------------------------------
const eventTypeColors = {
  "Community Meeting":      "#1f77b4",
  "Volunteer Training":     "#ff7f0e",
  "Rally":                  "#2ca02c",
  "Roundtable":             "#d62728",
  "Town Hall":              "#9467bd",
  "Canvassing Kickoff":     "#8c564b"
};

const eventTypeList = Object.keys(eventTypeColors);

// 2. Aggregate mean attendance by type
const avgAttendanceByType = Array.from(
  d3.rollups(
    lowIncomeEvents,
    v => d3.mean(v, d => d.estimated_attendance),
    d => d.event_type
  ),
  ([event_type, avg_attendance]) => ({
    event_type,
    avg_attendance
  })
).sort((a, b) => b.avg_attendance - a.avg_attendance); // descending

```
```js

//----------------------------------------------
// 3. Bar chart: attendance descending
//----------------------------------------------
const avgAttendancePlot = Plot.plot({
  title: "Average Attendance for Events in Low-Income Districts",
  height: 500,
  width: 800,
  marginLeft: 140,
  x: {
    label: "Event Type",
    domain: avgAttendanceByType.map(d => d.event_type),
  },
  y: {
    label: "Average Attendance"
  },
  color: {
    type: "ordinal",
    domain: eventTypeList,
    range: eventTypeList.map(t => eventTypeColors[t]),
    label: "Event Type"
  },
  marks: [
    Plot.barY(avgAttendanceByType, {
      x: "event_type",
      y: "avg_attendance",
      fill: d => eventTypeColors[d.event_type],
      tip: true,
      title: d =>
        `${d.event_type}
Average Attendance: ${d.avg_attendance.toFixed(1)}`
    }),
    Plot.text(avgAttendanceByType,{
      x: "event_type",
      y: "avg_attendance",
      text: (d) => d.avg_attendance.toFixed(0),
      textAnchor: "end",
      dx: 10,dy:18,
      fontWeight:"bold",
      fontSize:16
    })
  ]
});

display(avgAttendancePlot);

```

### We find that Roundtables and Volunteer Trainings are the top two events by average attendance, perhaps pointing to an opportunity to maximize these events when in said districts.

```js
//----------------------------------------------
// 1. Define survey alignment fields + nice labels
//----------------------------------------------
const issueFields = [
  { key: "affordable_housing_alignment", label: "Housing" },
  { key: "public_transit_alignment", label: "Transit" },
  { key: "childcare_support_alignment", label: "Childcare" },
  { key: "small_business_tax_alignment", label: "Small Biz Tax" },
  { key: "police_reform_alignment", label: "Police Reform" }
];


//----------------------------------------------
// 2. Compute average alignment per district per issue
//----------------------------------------------
const districtIssueAverages = Array.from(
  d3.rollups(
    survey,
    rows => {
      // compute mean per issue
      const avgs = issueFields.map(f => ({
        key: f.key,
        label: f.label,
        avg: d3.mean(rows, r => r[f.key])
      }));

      // choose the highest alignment issue
      const topIssue = avgs.reduce((a, b) => (b.avg > a.avg ? b : a));

      return topIssue;
    },
    d => d.boro_cd
  ),
  ([boro_cd, topIssue]) => ({
    boro_cd,
    top_issue_label: topIssue.label,
    top_issue_avg: topIssue.avg
  })
);


//----------------------------------------------
// 3. Join to district centroids
//----------------------------------------------
const districtIssueLabels = districtCentroids.map(dc => {
  const info = districtIssueAverages.find(d => d.boro_cd === dc.boro_cd);
  return {
    ...dc,
    top_issue_label: info?.top_issue_label ?? "",
    top_issue_avg: info?.top_issue_avg ?? 0
  };
});

```

<br>

## What Issues Can We Rely On Next Time? Especially in other Districts

```js
const topIssueMap = Plot.plot({
  title: "District’s Highest Policy Alignment (Post-Election Survey)",
  projection: {
    domain: districts,
    type: "mercator"
  },
  height: 720,
  width: 800,
  color: {
    type: "ordinal",
    domain: ["Low", "Middle", "High"],
    range: ["#d62828", "#fcd34d", "#16a34a"],
    label: "District Income Tier"
  },
  marks: [
    // --- Base polygons (effort score) ---
    Plot.geo(districts, {
      fill: d => {
        const key = d.properties.BoroCD;
        return incomeColors[districtStats.get(key)?.income];
      },
      stroke: "#f8f9fa",
      strokeWidth: 0.6,
      tip: false   // <-- no tooltips anywhere on this map
    }),

    // --- NEW: text at centroids showing highest-aligned issue ---
    Plot.text(districtIssueLabels, {
      x: d => d.coordinates[0],
      y: d => d.coordinates[1],
      text: d => d.top_issue_label,
      fontSize: 10,
      fill: "black",
      stroke: "white",
      strokeWidth: 1,
      textAnchor: "middle",
      tip: false
    })
  ]
});

display(topIssueMap);

```


```js
display(
  survey
    .filter(d => d.boro_cd === 501)
    // .slice(0, 20)
)


```