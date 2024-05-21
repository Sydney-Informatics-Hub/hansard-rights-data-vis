---
title: Types of rights by party
toc: false
theme: ["cotton", "wide"]
---

```js
const hansardRightsFile = FileAttachment("./data/hansard_rights.csv");
const timePeriodsFile = FileAttachment("./data/time_periods.csv");
const coerceHansardRow = (d) => ({"date": d.date, "party": (d.party === null ? "N/A" : d.party.toString()), "rights": (d.rights === null ? "" : d.rights.toString())});
const hansardRights = hansardRightsFile.csv({typed: true}).then((D) => D.map(coerceHansardRow));
const defaultTimePeriods = timePeriodsFile.csv({typed: true});
```

```js
var partySelection = ['All parties'];
var wordSelection = [];
var earliestDate = new Date(hansardRights.at(0).date.valueOf());
var latestDate = new Date(hansardRights.at(-1).date.valueOf());
for (let row of hansardRights) {
        if (!partySelection.includes(row.party)) partySelection.push(row.party);
    if (!wordSelection.includes(row.rights)) wordSelection.push(row.rights);
    if (row.date < earliestDate) earliestDate = new Date(row.date.valueOf());
    if (row.date > latestDate) latestDate = new Date(row.date.valueOf());
}
```



<div class="card">

<p>The hansard data presented on this page can be downloaded <a id="hansardDataLink" download>here</a></p>

```js
const hansardURL = await hansardRightsFile.url();

document.getElementById("hansardDataLink").href = hansardURL;
document.getElementById("hansardDataLink").download = "hansard_rights.csv";
```

</div>

<div class="card" >

## Date selector

<p>
Date ranges can be selected from the dropdown below or using the date picker. The default time periods are the active ministries throughout the time period covered by the data. You can <a id="timePeriodsLink" download>download the default time periods</a>, edit the CSV to include custom time periods, and re-upload below for convenience. Note: 1. the default will be reset upon reloading this page and 2. The dates in the file should conform to the format YYYY-MM-DD
</p>

```js
const timePeriodsURL = await timePeriodsFile.url();

document.getElementById("timePeriodsLink").href = timePeriodsURL;
document.getElementById("timePeriodsLink").download = "time_periods.csv";
```

```js
const userTimePeriodsFile = view(Inputs.file({label: "Custom time periods", accept: ".csv"}));
```

```js
const userTimePeriods = userTimePeriodsFile ? userTimePeriodsFile.csv({typed: true}) : null;
```

```js
const selectedTimePeriod = view(Inputs.select(userTimePeriods ? userTimePeriods : defaultTimePeriods, {label: "Time periods", format: (d) => (d.name) }));
```

```js
const startDate = view(Inputs.date({label: "Start date range", value: new Date(d3.max([selectedTimePeriod.start_date.valueOf(), earliestDate.valueOf()])), min: new Date(earliestDate.valueOf()), max: latestDate, required: true}));
const endDate = view(Inputs.date({label: "End date range", value: new Date(d3.min([selectedTimePeriod.end_date.valueOf(), latestDate.valueOf()])), min: new Date(earliestDate.valueOf()), max: latestDate, required: true}));
```

</div>

<div class="card" >

## Types of rights mentioned by count, filtered by date and party

```js
const wordsInput = Inputs.select(wordSelection, {label: "Rights", sort: true, value: ["human"], multiple: 20 });
const words = view(wordsInput);
```

```js
const party = view(Inputs.select(partySelection, {label: "Party"}));
```

```js
const hansardRightsFiltered = [];
for (let row of hansardRights) {
    if ((words.includes(row.rights)) && ((party === 'All parties') || (row.party === party)) && (new Date(row.date) >= startDate) && (new Date(row.date) <= endDate)) {
        hansardRightsFiltered.push(row);
    }
}
```

```js
function stringToColour(str) {
    if (str === null) {
        str = "";
    }
    let hash = 0;
    str.split('').forEach(char => {
        hash = char.charCodeAt(0) + ((hash << 5) - hash);
    })
    let colour = '#';
    for (let i = 0; i < 3; i++) {
        const value = (hash >> (i * 8)) & 0xff;
        colour += value.toString(16).padStart(2, '0');
    }
    return colour;
}
```

```js
display(Plot.plot({
    subtitle: `Types of rights mentioned by ${party.toLowerCase()} between ${startDate.toDateString()} and ${endDate.toDateString()}`,
    height: 40 + new Set(hansardRightsFiltered.map(d => d.rights)).size * 20,
    width: width,
    marginLeft: 100,
    marginRight: 40,
    x: {
        grid: true
    },
    color: {
        type: "categorical",
        scheme: "Tableau10"
    },
    marks: [
        Plot.barX(hansardRightsFiltered, Plot.groupY({ x: "count" }, { y: "rights", fill: (d) => stringToColour(d.rights), sort: { y: "x", reverse: true } })),
        Plot.text(hansardRightsFiltered, Plot.groupY({ x: "count", text: "count" }, { y: "rights", sort: { y: "x", reverse: true }, frameAnchor: "left", dx: 3 }))
    ]
}));
```

</div>

<div class="card" >

```js
const activeParties = view(Inputs.select(partySelection.slice(1), { label: "Parties", value: ["Lab", "Con"], multiple: 3 }));
```

```js
const wordsSingle = view(Inputs.select(wordSelection, { label: "Rights", sort: true, value: "human", size: 20 }));
```

```js
const hansardRightsSingleWord = hansardRights.filter((d) => (d.rights == wordsSingle));

function padInt(intValue, numPadChars) {
    return intValue.toString().padStart(numPadChars, '0');
}

const countMap = new Map();
let entryDate;
let formattedDate;
let currKey;
for (let currParty of partySelection) {
    entryDate = new Date(earliestDate.valueOf());
    while (entryDate <= latestDate) {
        let formattedDate = `${padInt(entryDate.getUTCFullYear(), 4)}-${padInt(entryDate.getMonth()+1, 2)}-${padInt(entryDate.getDate(), 2)}`
        currKey = `${formattedDate}:${currParty}`;
        countMap.set(currKey, 0);
        entryDate.setDate(entryDate.getDate() + 1);
    }
}
for (let row of hansardRightsSingleWord) {
    formattedDate = `${padInt(row.date.getUTCFullYear(), 4)}-${padInt(row.date.getMonth()+1, 2)}-${padInt(row.date.getDate(), 2)}`
    currKey = `${formattedDate}:${row.party}`;
    if (countMap.has(currKey)) {
        countMap.set(currKey, countMap.get(currKey) + 1);
    } else {
        countMap.set(currKey, 1);
    }
}

const hansardCounts = [];
let currDateStr;
let currDate;
let currParty;
let rowObj;
for (const [key, value] of countMap) {
    currDateStr = key.split(":").at(0);
    currDate = new Date(currDateStr);
    if ((currDate < startDate) || (currDate > endDate)) {
        continue;
    }
    currParty = key.split(":").at(1);
    rowObj = {date: currDateStr, party: currParty, count: value};
    hansardCounts.push(rowObj);
}
```

```js
const windowK = view(Inputs.range([1, 730], { value: 365, step: 1, label: "Rolling average window (days)"}));
```

```js
const meanSumToggle = view(Inputs.radio(["mean", "sum"], {label: "Rolling mean/rolling sum", value: "mean"}));
```

```js
display(Plot.plot({
    title: `"${wordsSingle}" rights mentioned by party on a rolling ${windowK} day count`,
    width: width,
    x: {
        type: "time",
        grid: true
    },
    y: {
        label: meanSumToggle
    },
    color: {
        type: "categorical",
        scheme: "Set2",
        domain: activeParties,
        legend: true
    },
    marks: [
        Plot.lineY(hansardCounts, Plot.windowY({ k: windowK, reduce: meanSumToggle, x: "date", y: "count", stroke: "party", tip: true }))
    ]
}));
```

</div>
