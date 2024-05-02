---
title: Types of rights by party
toc: false
---

```js
const hansardRights = FileAttachment("./data/hansard-rights.csv").csv({typed: true});
```

```js
var partySelection = [];
hansardRights.forEach((d) => {
    if (!partySelection.includes(d.party)) partySelection.push(d.party);
});
const party = view(Inputs.select(partySelection, {label: "Party"}));
```

Set a minimum count filter below. Words that never have a count greater than the minimum for any year won't appear. Set to 0 for no minimum.
```js
const perYearCountMin = view(Inputs.text({label: "Per year count min", type: 'number', value: 0, min: 0, required: true}));
```
Set a maximum count filter below. Words that have at least one count greater than the maximum for any year won't appear. Leave blank for no limit.
```js
const perYearCountMax = view(Inputs.text({label: "Per year count max", type: 'number', value: '', min: 0, required: false}));
```

```js
const initialPartyFilter = hansardRights.filter(row => (row.party === party));
const yearsCovered = [];
hansardRights.forEach((d) => {
    if (!yearsCovered.includes(d.year)) yearsCovered.push(d.year);
});
const includeList = [];
const overThresholdWords = [];
yearsCovered.forEach((year) => {
    var sorted = d3.sort(initialPartyFilter.filter(row => (row.year === year)), (d) => d.count);
    sorted.forEach((d) => {
        if (!includeList.includes(d.word) && (d.word !== 'human') && (d.count >= perYearCountMin)) includeList.push(d.word);
        if ((perYearCountMax != '') && !overThresholdWords.includes(d.word) && (d.count > perYearCountMax)) overThresholdWords.push(d.word);
    });
});
const wordList = [];
includeList.forEach((word) => {
    if (!overThresholdWords.includes(word)) wordList.push(word);
});

const hansardRightsFiltered = initialPartyFilter.filter(row => (wordList.includes(row.word)));
const hansardHumanRights = initialPartyFilter.filter(row => (row.word === "human"));
```

```js
const rightsComparisonPartyPlot = Plot.plot({
    title: `Type of rights mentions by the ${party} party`,
    subtitle: "Types of rights mentioned over time, sorted by first mention",
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
    title: `Human rights mentions by the ${party} party`,
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

<div class="card" >
    ${rightsComparisonPartyPlot}
</div>

<div class="card" >
    ${humanRightsPartyPlot}
</div>