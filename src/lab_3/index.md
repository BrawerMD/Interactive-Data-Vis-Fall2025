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
      strokeWidth: 0.3,
    }),
  ]
});

display(effortScoreMap);
```

