---
title: "Lab 2: Subway Staffing"
toc: true
---

# Planning for 2026 with the MTA

## We learned that a subway fare increase may relate to lower ridership

To begin. our data adventure, we approached average monthly ridership (defined as entrances into our stations) over the summer. We first look at all stations, then try to clean up a bit by grouping them into categories:

*Author's note: an improvement would be to normalize to 7 day weeks so as to have cleaner time aggregations that do not partially encompass the fare change.*

```js
const incidents = FileAttachment("./data/incidents.csv").csv({ typed: true })
const local_events = FileAttachment("./data/local_events.csv").csv({ typed: true })
const upcoming_events = FileAttachment("./data/upcoming_events.csv").csv({ typed: true })
const ridership = FileAttachment("./data/ridership.csv").csv({ typed: true })
```


```js
const currentStaffing = {
  "Times Sq-42 St": 19,
  "Grand Central-42 St": 18,
  "34 St-Penn Station": 15,
  "14 St-Union Sq": 4,
  "Fulton St": 17,
  "42 St-Port Authority": 14,
  "Herald Sq-34 St": 15,
  "Canal St": 4,
  "59 St-Columbus Circle": 6,
  "125 St": 7,
  "96 St": 19,
  "86 St": 19,
  "72 St": 10,
  "66 St-Lincoln Center": 15,
  "50 St": 20,
  "28 St": 13,
  "23 St": 8,
  "Christopher St": 15,
  "Houston St": 18,
  "Spring St": 12,
  "Chambers St": 18,
  "Wall St": 9,
  "Bowling Green": 6,
  "West 4 St-Wash Sq": 4,
  "Astor Pl": 7
}
```

```js
const monthlyAvg = Array.from(
  d3.rollup(
    ridership,
    v => d3.mean(v, d => d.entrances ?? 0),
    d => d3.timeMonth(d.date),
    d => d.station
  ),
  ([date, stations]) =>
    Array.from(stations, ([station, avg]) => ({ date, station, avg }))
).flat();

const stationGroups = {
  "Major Hubs": [
    "Times Sq-42 St",
    "Grand Central-42 St",
    "34 St-Penn Station",
    "42 St-Port Authority"
  ],
  "Midtown": [
    "Herald Sq-34 St",
    "59 St-Columbus Circle",
    "50 St",
    "66 St-Lincoln Center"
  ],
  "Downtown": [
    "Fulton St",
    "Chambers St",
    "Wall St",
    "Bowling Green",
    "Canal St"
  ],
  "Local": [
    "14 St-Union Sq",
    "West 4 St-Wash Sq",
    "Astor Pl",
    "Spring St",
    "Houston St",
    "23 St",
    "28 St",
    "72 St",
    "86 St",
    "96 St"
  ]
};

function getGroup(station) {
  for (const [group, list] of Object.entries(stationGroups)) {
    if (list.includes(station)) return group;
  }
  return "Other";
}

const monthlyGrouped = monthlyAvg.map(d => ({
  ...d,
  group: getGroup(d.station)
}));

const groupedMonthly = Array.from(
  d3.rollup(
    monthlyGrouped,
    v => d3.mean(v, d => d.avg),
    d => d3.timeMonth(d.date),
    d => d.group
  ),
  ([date, groups]) =>
    Array.from(groups, ([group, avg]) => ({ date, group, avg }))
).flat();


display(
  Plot.plot({
    title: "Average Monthly Entrances by Station (Normalized to First Month)",
    height: 500,
    x: { label: "Month", type: "time",      domain: [new Date("2025-05-01"), new Date("2025-08-31")] },
    y: {
      label: "Change in Ridership (%)",
      tickFormat: ((f) => (x) => f((x - 1) * 100))(d3.format("+.0f"))
    },
    color: { legend: true, label: "Station" },
    marks: [
      Plot.ruleY([1]),
      Plot.line(monthlyAvg, Plot.normalizeY("first", {
        x: "date",
        y: "avg",
        stroke: "station"
      })),
      Plot.text(
        monthlyAvg,
        Plot.selectLast(Plot.normalizeY("first", {
          x: "date",
          y: "avg",
          z: "station",
          text: "station",
          textAnchor: "start",
          dx: 3
        }))
      )
    ]
  })
);

display(
  Plot.plot({
    title: "Average Monthly Entrances by Station Group",
    height: 500,
    x: { label: "Month", type: "time" },
    y: { label: "Average Entrances" },
    color: { legend: true, label: "Group" },
    marks: [
      Plot.line(groupedMonthly, { x: "date", y: "avg", stroke: "group" }),
      Plot.dot(groupedMonthly, {
        x: "date",
        y: "avg",
        stroke: "group",
        fill: "group",
        symbol: "diamond",
        r: 4,
        tip: true,
        title: d =>
          `${d.group} – ${d3.timeFormat("%B")(d.date)}\nAvg Entrances: ${d3.format(",.0f")(d.avg)}`
      }),
      Plot.ruleX([new Date("2025-07-15")], {
        stroke: "red",
        strokeDasharray: "4,2",
        strokeWidth: 2,
        label: "Fare Increase",
        dy: -10
      }),
      Plot.text(
      [{y: 15000, x: new Date("2025-07-15"), text: "July Fare Increase"}],
      {
        x: "x",
        y: "y",
        text: "text",
        fontWeight: "bold",
        fill: "#ffffffff",
        textAnchor: "middle",
        dy: -5
      }
    )
    ]
  })
);

```

These data show us that ridership declines all summer, which makes sense! New Yorkers don't need this heat. Looking at the slow of our lines, we find that most stations were already seeing a marked decline before the 7/15 fare increase, and the increase did not accelerate any declines. 

We cannot infer causality, but this points us toward a conclusion that the fare icnrease alone did not change behavior. With that said, more ideal inference would compare the same months year over year to account for this seasonality.

## Could events be driving the change? How?

We observe major events raising ridership numbers by aggregating event days and non-event days. A popular station like Union Square may get the most % change from a HUGE event (see the top most bubble in chart 2), whereas even a smaller event like the Fireworks display at West 4th can raise the ridership by nearly 100% as a function of usual ridership:

```js
const events = local_events.map(e => ({
  date: e.date,
  station: e.nearby_station,
  attendance: e.estimated_attendance
}));

const eventAgg = Array.from(
  d3.rollup(
    events,
    v => d3.sum(v, d => d.attendance ?? 0),
    d => d3.timeDay(d.date),
    d => d.station
  ),
  ([date, stations]) => Array.from(stations, ([station, totalAttendance]) => ({
    date,
    station,
    totalAttendance
  }))
).flat();

const ridershipWithEvents = ridership.map(r => {
  const match = eventAgg.find(
    e => e.station === r.station && +e.date === +d3.timeDay(r.date)
  );
  return {
    date: r.date,
    station: r.station,
    entrances: r.entrances ?? 0,
    exits: r.exits ?? 0,
    totalAttendance: match ? match.totalAttendance : 0
  };
});

const ridershipEventImpact = ridershipWithEvents.map(r => ({
  ...r,
  totalRiders: (r.entrances ?? 0) + (r.exits ?? 0),
  eventDay: r.totalAttendance > 0 ? "Event Day" : "No Event"
}));

const eventComparison = Array.from(
  d3.rollup(
    ridershipEventImpact,
    v => ({
      avgEvent: d3.mean(v.filter(d => d.eventDay === "Event Day"), d => d.totalRiders),
      avgNonEvent: d3.mean(v.filter(d => d.eventDay === "No Event"), d => d.totalRiders)
    }),
    d => d.station
  ),
  ([station, vals]) => ({
    station,
    avgEvent: vals.avgEvent,
    avgNonEvent: vals.avgNonEvent,
    pctChange: vals.avgEvent && vals.avgNonEvent
      ? ((vals.avgEvent - vals.avgNonEvent) / vals.avgNonEvent) * 100
      : null
  })
);



display(
  Plot.plot({
    title: "Event-Day Ridership Impact by Station",
    marginLeft: 140,
    height: 600,
    x: { label: "% Change vs. Non-Event Days", tickFormat: d3.format("+.0f") },
    y: { label: "Station", domain: eventComparison.sort((a,b)=>b.pctChange - a.pctChange).map(d => d.station) },
    marks: [
      Plot.ruleX([0]),
      Plot.barX(eventComparison, {
        x: "pctChange",
        y: "station",
        fill: d => d.pctChange > 0 ? "#57bb8a" : "#e15759",
        title: d =>
          `${d.station}\nEvent-Day Avg: ${d3.format(",")(d.avgEvent ?? 0)}\nNon-Event Avg: ${d3.format(",")(d.avgNonEvent ?? 0)}\nChange: ${d3.format("+.1f")(d.pctChange ?? 0)}%`
      })
    ]
  })
);
```
```js
const ridershipByDayStation = d3.rollup(
  ridership,
  v => d3.mean(v, d => (d.entrances ?? 0) + (d.exits ?? 0)),
  d => d3.timeDay(d.date),
  d => d.station
);

const baseRidershipByStation = d3.rollup(
  ridership,
  v => d3.mean(v, d => (d.entrances ?? 0) + (d.exits ?? 0)),
  d => d.station
);

const eventImpactPoints = local_events.map(e => {
  const day = d3.timeDay(e.date);
  const base = baseRidershipByStation.get(e.nearby_station) ?? 0;
  const eventDay = ridershipByDayStation.get(day)?.get(e.nearby_station) ?? base;
  const uplift = base ? (eventDay - base) / base : 0;
  return {
    date: e.date,
    event_name: e.event_name,
    station: e.nearby_station,
    attendance: e.estimated_attendance ?? 0,
    ridership_base: base,
    ridership_event: eventDay,
    uplift,
    upliftPct: uplift * 100
  };
});

// --- Scatter plot: Attendance vs Ridership Change ---
display(
  Plot.plot({
    title: "Event Attendance vs. Ridership Increase",
    height: 500,
    grid: true,
    x: {
      label: "Estimated Attendance",
      tickFormat: d3.format(",")
    },
    y: {
      label: "Ridership % Change vs Baseline",
      tickFormat: d => `${d}%`
    },
    color: { label: "Nearby Station" },
    marks: [
      Plot.text(
      [{y: 120, x: 9500, text: "Gallery Opening 14th St"}],
      {
        x: "x",
        y: "y",
        text: "text",
        fontWeight: "bold",
        fill: "#ffffffff",
        textAnchor: "middle",
        dy: -5
      }
    ),
    Plot.text(
      [{y: 90, x: 1850, text: "Fireworks W4th"}],
      {
        x: "x",
        y: "y",
        text: "text",
        fontWeight: "bold",
        fill: "#ffffffff",
        textAnchor: "middle",
        dy: -5
      }
    ),
      Plot.dot(eventImpactPoints, {
        x: "attendance",
        y: "upliftPct",
        r: d => Math.abs(d.upliftPct) / 2 + 3,
        fill: "station",
        stroke: "station",
        fillOpacity: 0.7,
        tip: true,
        title: d =>
          `${d.event_name}\nStation: ${d.station}\nAttendance: ${d3.format(",")(d.attendance)}\nRidership Change: ${d3.format("+.1f")(d.upliftPct)}%`
      }),
      Plot.ruleY([0], { stroke: "#888", strokeDasharray: "4,2" })
    ]
  })
);
```


# Up next, let's see the response time to incidents by station. 

We take an average for incidents at each stationa and compare to the overall average of 10.29 Minutes. 

We find **Fulton St.** to be the quickest at response times and **West 4th to be the slowest**:
```js
const severityWeights = { low: 1, medium: 2, high: 3 };

const responseStats = Array.from(
  d3.rollup(
    incidents,
    v => {
      const times = v.map(d => d.response_time_minutes ?? 0);
      const weights = v.map(d => severityWeights[(d.severity || "").toLowerCase()] ?? 1);
      const weightedMean =
        d3.sum(times.map((t, i) => t * weights[i])) / d3.sum(weights);
      const median = d3.median(times);
      const p90 = d3.quantile(times.sort(d3.ascending), 0.9);
      return {
        weightedMean,
        median,
        p90,
        nIncidents: v.length
      };
    },
    d => d.station
  ),
  ([station, vals]) => ({ station, ...vals })
).sort((a, b) => d3.ascending(a.weightedMean, b.weightedMean));


display(
  Plot.plot({
    title: "Average (Severity-Weighted) Response Time by Station",
    marginLeft: 140,
    height: 600,
    x: { label: "Avg Response Time (minutes)" },
    y: { label: "Station", domain: responseStats.map(d => d.station) },
    color: {
      type: "linear",
      scheme: "RdYlGn",
      reverse: true,
      legend: true,
      label: "Response Time (min)"
    },
    marks: [
      Plot.barX(responseStats, {
        x: "weightedMean",
        y: "station",
        fill: "weightedMean",
        title: d =>
          `${d.station}\nWeighted Avg: ${d3.format(".1f")(d.weightedMean)} min\nMedian: ${d3.format(".1f")(d.median)} min\n90th %ile: ${d3.format(".1f")(d.p90)} min\nIncidents: ${d.nIncidents}`
      }),
      //def redundant to be using ruleX and text, but have to understand whats wrong there (todo!)
      Plot.ruleX([d3.mean(responseStats, d => d.weightedMean)], {
        stroke: "black",
        strokeDasharray: "4,2",
        label: "Overall Avg",
        dy: -10
      }),
      Plot.text(responseStats, {
        x: "weightedMean",
        y: "station", text: "weightedMean"}),
      Plot.text(
      [{y: "50 St", x: [d3.mean(responseStats, d => d.weightedMean)], text: "Overall Response Avg: 10.3"}],
      {
        x: "x",
        y: "y",
        text: "text",
        fontWeight: "bold",
        fill: "#ffffffff",
        textAnchor: "middle",
        dy: -5
      }
    ),
    ]
  })
);
```

# Staffing 
In preparation for the next question, we got to thinking: how about we normalize per staff member?

```js
const severityWeights = { low: 1, medium: 2, high: 3 };

const responseStats = Array.from(
  d3.rollup(
    incidents,
    v => {
      const times = v.map(d => d.response_time_minutes ?? 0);
      const weights = v.map(d => severityWeights[(d.severity || "").toLowerCase()] ?? 1);
      const staff = v.map(d => d.staffing_count ?? 1);

      // Normalize each response time per staff member, then apply severity weighting
      const perStaffTimes = times.map((t, i) => t / staff[i]);
      const weightedMean =
        d3.sum(perStaffTimes.map((t, i) => t * weights[i])) / d3.sum(weights);

      const median = d3.median(perStaffTimes);
      const p90 = d3.quantile(perStaffTimes.sort(d3.ascending), 0.9);
      return {
        weightedMean,
        median,
        p90,
        nIncidents: v.length
      };
    },
    d => d.station
  ),
  ([station, vals]) => ({ station, ...vals })
).sort((a, b) => d3.ascending(a.weightedMean, b.weightedMean));

// --- Visualization: Response Time Ranking ---
display(
  Plot.plot({
    title: "Average (Severity-Weighted, Per-Staff) Response Time by Station",
    marginLeft: 140,
    height: 600,
    x: { label: "Avg Response Time per Staff (minutes)" },
    y: { label: "Station", domain: responseStats.map(d => d.station) },
    color: {
      type: "linear",
      scheme: "RdYlGn",
      domain: d3.extent(responseStats, d => d.weightedMean),
      reverse: true,
      legend: true,
      label: "Response Time per Staff (min)"
    },
    marks: [
      Plot.barX(responseStats, {
        x: "weightedMean",
        y: "station",
        fill: "weightedMean",
        title: d =>
          `${d.station}\nWeighted Avg (per staff): ${d3.format(".1f")(d.weightedMean)} min\nMedian: ${d3.format(".1f")(d.median)} min\n90th %ile: ${d3.format(".1f")(d.p90)} min\nIncidents: ${d.nIncidents}`
      }),
      Plot.ruleX([d3.mean(responseStats, d => d.weightedMean)], {
        stroke: "black",
        strokeDasharray: "4,2",
        label: "Overall Avg",
        dy: -10
      })
    ]
  })
);
```

RIP West 4th - something about this station, as well as Union Square and Canal, seems to point to a need for extra staffing help because response time is slowest given their usual staffing numbers.

To project forward to 2026, we can leverage future events data and combine it with incident rssponse timing to anticipate staffing needs.

```js
// --- 1. Aggregate total event attendance for 2025 and 2026 ---
const events25 = local_events.map(e => ({
  date: e.date,
  station: e.nearby_station,
  attendance: e.estimated_attendance ?? 0
}));

const events26 = upcoming_events.map(e => ({
  date: e.date,
  station: e.nearby_station,
  attendance: e.expected_attendance ?? 0
}));

const attendance25 = Array.from(
  d3.rollup(events25, v => d3.sum(v, d => d.attendance), d => d.station),
  ([station, total]) => ({ station, total2025: total })
);

const attendance26 = Array.from(
  d3.rollup(events26, v => d3.sum(v, d => d.attendance), d => d.station),
  ([station, total]) => ({ station, total2026: total })
);

// --- 2. Join both years and calculate change ---
function joinEvents(att25, att26) {
  const map26 = new Map(att26.map(d => [d.station, d]));
  return att25.map(d => {
    const match = map26.get(d.station);
    const total26 = match ? match.total2026 : 0;
    const diff = total26 - d.total2025;
    const pct = d.total2025 > 0 ? (diff / d.total2025) * 100 : null;
    return {
      station: d.station,
      total2025: d.total2025,
      total2026: total26,
      diff,
      pct
    };
  });
}

const joinedEvents = joinEvents(attendance25, attendance26);

// --- 3. Compute per-staff response efficiency from incidents ---
const severityWeights = { low: 1, medium: 2, high: 3 };
const responsePerStaff = Array.from(
  d3.rollup(
    incidents,
    v => {
      const times = v.map(d => d.response_time_minutes ?? 0);
      const weights = v.map(d => severityWeights[(d.severity || "").toLowerCase()] ?? 1);
      const staff = v.map(d => d.staffing_count ?? 1);
      const perStaff = times.map((t, i) => t / staff[i]);
      return d3.sum(perStaff.map((t, i) => t * weights[i])) / d3.sum(weights);
    },
    d => d.station
  ),
  ([station, perStaffMean]) => ({ station, perStaffMean })
);

// --- 4. Predict staffing needs for 2026 ---
const staffingProjections = joinedEvents.map(d => {
  const resp = responsePerStaff.find(r => r.station === d.station);
  const perStaffTime = resp?.perStaffMean ?? 10;
  const current = currentStaffing[d.station] ?? 0;

  // Assume attendance growth scales with required staffing
  const ratio = d.total2025 > 0 ? d.total2026 / d.total2025 : 1;
  const projectedStaff = current * ratio;
  const addlNeeded = Math.max(0, Math.round(projectedStaff - current));

  // Priority score = added staff × response inefficiency factor
  const priorityScore = addlNeeded * (perStaffTime / 10);

  return {
    station: d.station,
    currentStaff: current,
    projectedStaff: Math.round(projectedStaff),
    addlNeeded,
    perStaffTime: perStaffTime.toFixed(1),
    pctAttendanceChange: d.pct,
    priorityScore: +priorityScore.toFixed(2)
  };
}).sort((a,b)=>d3.descending(a.priorityScore,b.priorityScore));

// --- 5. Visualization: projected change in attendance ---
display(
  Plot.plot({
    title: "Change in Projected Event Attendance by Station (2026 vs 2025)",
    marginLeft: 150,
    height: 600,
    x: { label: "% Change in Attendance", tickFormat: d3.format("+.0f") },
    y: { label: "Station", domain: joinedEvents.sort((a,b)=>b.pct - a.pct).map(d => d.station) },
    color: {
      type: "diverging",
      scheme: "RdYlGn",
      domain: [d3.min(joinedEvents, d => d.pct), d3.max(joinedEvents, d => d.pct)],
      legend: true,
      label: "% Change"
    },
    marks: [
      Plot.ruleX([0]),
      Plot.barX(joinedEvents, {
        x: "pct",
        y: "station",
        fill: "pct",
        title: d =>
          `${d.station}\n2025 Attendance: ${d3.format(",")(d.total2025 ?? 0)}\n2026 Attendance: ${d3.format(",")(d.total2026 ?? 0)}\nChange: ${d3.format("+.1f")(d.pct ?? 0)}%`
      })
    ]
  })
);
```

This tells us that Fulton, West 4th, and 23rd are anticipating the most event-based growth next year.

## Bonus: Picking One?
I said it once and I'll say it again: RIP West 4th, the only entrant to appear in both the bottom of our historical response data (to incidents) AND to be seeing a major event attendance increase next year.


# But Wait: There's More
Upon further review we note that Canal St. and Times Square are not in the local_events data, meaning that our approach to looking specifically at growth may ignore these stations.

For good measure let's look at current staffing vs. Future Events:



```js
// Sum attendance for Canal St in 2025 and 2026
const total2025 = d3.sum(local_events.filter(e => e.nearby_station === "Canal St"), e => e.estimated_attendance ?? 0);
const total2026 = d3.sum(upcoming_events.filter(e => e.nearby_station === "Canal St"), e => e.expected_attendance ?? 0);

console.log("Canal St - Total Attendance 2025:", total2025);
console.log("Canal St - Total Attendance 2026:", total2026);
console.log("Change (2026 - 2025):", total2026 - total2025);

// Loop through all nearby_station entries and log total 2025 attendance per station
const totals2025 = Array.from(
  d3.rollup(
    local_events,
    v => d3.sum(v, d => d.estimated_attendance ?? 0),
    d => d.nearby_station
  ),
  ([station, total]) => ({ station, total })
);

for (const { station, total } of totals2025) {
  console.log(`${station}: ${total}`);
}

```

```js
// --- Sum total 2026 event attendance by station ---
const attendance26 = Array.from(
  d3.rollup(
    upcoming_events,
    v => d3.sum(v, d => d.expected_attendance ?? 0),
    d => d.nearby_station
  ),
  ([station, total]) => ({ station, totalAttendance: total })
);

// --- Combine with current staffing ---
const staffCoverage = attendance26.map(d => {
  const staff = currentStaffing[d.station] ?? 0;
  const attendeesPerStaff = staff > 0 ? d.totalAttendance / staff : null;
  return {
    station: d.station,
    totalAttendance: d.totalAttendance,
    staff,
    attendeesPerStaff
  };
}).filter(d => d.attendeesPerStaff !== null);

// --- Sort from best coverage (fewest attendees per staff) to worst ---
staffCoverage.sort((a, b) => d3.ascending(a.attendeesPerStaff, b.attendeesPerStaff));

// --- Visualization: Staff Coverage Ratio ---
display(
  Plot.plot({
    title: "2026 Event Attendees per Staff Member by Station",
    marginLeft: 150,
    height: 600,
    x: {
      label: "Attendees per Staff Member",
      tickFormat: d3.format(",")
    },
    y: {
      label: "Station",
      domain: staffCoverage.map(d => d.station)
    },
    color: {
      type: "linear",
      scheme: "YlOrRd",
      domain: d3.extent(staffCoverage, d => d.attendeesPerStaff),
      reverse: true,
      legend: true,
      label: "Attendees per Staff"
    },
    marks: [
      Plot.ruleX([0]),
      Plot.barX(staffCoverage, {
        x: "attendeesPerStaff",
        y: "station",
        fill: "attendeesPerStaff",
        title: d =>
          `${d.station}\nTotal 2026 Attendance: ${d3.format(",")(d.totalAttendance)}\nStaff: ${d.staff}\nAttendees per Staff: ${d3.format(",.0f")(d.attendeesPerStaff)}`
      })
    ]
  })
);

```

Ok, twist my arm. Because Canal has unprecedented growth to expect and far and away the most inadequate coverage of staffing to attendees, let's call that our **number one staffing increase choice.**