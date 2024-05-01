---
title: Overview of all parties
toc: false
---

```js
const hansardRights = FileAttachment("./data/hansard-rights.csv").csv({typed: true});
```

```js
const yearsCovered = [];
hansardRights.forEach((d) => {
    if (!yearsCovered.includes(d.year)) yearsCovered.push(d.year);
});
const hansardGrouped = [];
var wordMap;
yearsCovered.forEach((year) => {
    wordMap = new Map();
    hansardRights.filter(row => (row.year === year)).forEach((row) => {
        if (wordMap.has(row.word)) {
            wordMap.set(row.word, wordMap.get(row.word) + row.count);
        } else {
            wordMap.set(row.word, row.count);
        }
    });
    wordMap.forEach((count, word) => {
        hansardGrouped.push({year: year, word: word, count: count});
    });
});
```

```js
const yearsCovered = [];
hansardRights.forEach((d) => {
    if (!yearsCovered.includes(d.year)) yearsCovered.push(d.year);
});
const wordList = [];
yearsCovered.forEach((year) => {
    var sorted = d3.sort(hansardGrouped.filter(row => (row.year === year)), (d) => d.count);
    sorted.slice(-5).forEach((d) => {
        if (!wordList.includes(d.word) && (d.word !== 'human') && (d.count > 1)) wordList.push(d.word);
    });
});
const hansardRightsFiltered = hansardGrouped.filter(row => (wordList.includes(row.word)));
const hansardHumanRights = hansardGrouped.filter(row => (row.word === "human"));
```

```js
const rightsComparisonPartyPlot = Plot.plot({
    title: "Type of rights mentions by all parties",
    subtitle: "Types of rights mentioned over time, sorted by first mention. Rights are listed only if they are in the top 5 by count and have a count of more than 1 for at least one year",
    height: d3.max([40 + new Set(hansardRightsFiltered.map(d => d.word)).size * 20, 600]),
    width,
    marginBottom: 100,
    marginLeft: 120,
    x: {grid: true},
    y: {axis: null, range: [40, -10]},
    fy: {label: null, domain: hansardRightsFiltered.map(d => d.word)},
    color: {
        type: "categorical",
        scheme: "Tableau10"
    },
    marks: [
        Plot.axisX({anchor: "top", label: "Year", labelAnchor: "center", labelArrow: "none", tickPadding: 10, labelOffset: 35, tickFormat: (d) => (d.toString())}),
        Plot.areaY(hansardRightsFiltered, {x: "year", y: "count", fy: "word", curve: "monotone-x", fill: "word", fillOpacity: 0.5}),
        Plot.lineY(hansardRightsFiltered, {x: "year", y: "count", fy: "word", curve: "monotone-x", strokeWidth: 0.8, tip: true, title: (d) => `Count: ${d.count}\nWord: ${d.word}\nYear: ${d.year}`})
    ]
});
```

```js
const humanRightsPartyPlot = Plot.plot({
    title: "Human rights mentions by all parties",
    width,
    marginTop: 50,
    marginBottom: 100,
    marginLeft: 120,
    x: {grid: true},
    marks: [
        Plot.axisY({anchor: "left", label: "Count", labelAnchor: "center", labelArrow: "none", labelOffset: 70}),
        Plot.axisX({anchor: "top", label: "Year", labelAnchor: "center", labelArrow: "none", tickPadding: 10, labelOffset: 35, tickFormat: (d) => (d.toString())}),
        Plot.areaY(hansardHumanRights, {x: "year", y: "count",  curve: "monotone-x", fill: "word", fillOpacity: 0.3}),
        Plot.lineY(hansardHumanRights, {x: "year", y: "count", curve: "monotone-x", strokeWidth: 1, tip: 'x', title: (d) => `Count: ${d.count}\nYear: ${d.year}`})
    ]
});
```

```js
const hansardAllHumanRights = hansardRights.filter(row => (row.word === "human"));
```

```js
const humanRightsPlot = Plot.plot({
    title: "Comparison of number of mentions of human rights by party",
    subtitle: "Human rights mentioned by party over time, sorted by first mention",
    height: 700,
    width,
    marginTop: 50,
    marginBottom: 100,
    marginLeft: 120,
    x: {grid: true},
    y: {axis: null, range: [70, 20]},
    fy: {label: null, domain: hansardAllHumanRights.map(d => d.party)},
    marks: [
        Plot.axisX({anchor: "bottom", label: "Year", labelAnchor: "center", labelArrow: "none", tickPadding: 10, labelOffset: 35, tickFormat: (d) => (d.toString())}),
        Plot.axisX({anchor: "top", label: "", labelArrow: "none", tickPadding: 30, labelOffset: 35, tickFormat: (d) => (d.toString())}),
        Plot.areaY(hansardAllHumanRights, {x: "year", y: "count", fy: "party", curve: "basis", fill: "party", fillOpacity: 0.3}),
        Plot.lineY(hansardAllHumanRights, {x: "year", y: "count", fy: "party", curve: "basis", strokeWidth: 0.8, tip: 'xy', title: (d) => `Count: ${d.count}\nParty: ${d.party}\nYear: ${d.year}`})
    ]
});
```

<div class="card" >
    ${rightsComparisonPartyPlot}
</div>

<div class="card" >
    ${humanRightsPartyPlot}
</div>

<div class="card" >
    ${humanRightsPlot}
</div>