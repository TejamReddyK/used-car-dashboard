<style>
  .content-section {
    width: 100%;
    max-width: 1200px; /* match visualization width */
    margin: 0 auto;
    padding: 10px 20px;
    box-sizing: border-box;
  }

  .content-section h3 {
    margin-top: 20px;
    font-size: 20px;
  }

  .content-section p,
  .content-section ul {
    font-size: 16px;
    line-height: 1.6;
    margin: 0;
  }
</style>


---
<div>
  <h3>Tejam Reddy Kalam</h3>
  <h3>Student ID: 801412573</h3>
</div>

<div style="text-align: center; max-width: 800px; margin: 0 auto;">
  <h2 style="margin: 10px 0;">Car Sales Analysis Dashboard</h2>
  <p>This dashboard explores trends in car sales by analyzing sale price, salesperson commissions, and performance across car makes and years.</p>
</div>

---
<div class="content-section">
  <h3>🚗 Theme</h3>
  <p>
    Understanding car sales trends, pricing patterns, and salesperson performance
    using commission and volume indicators.
  </p>
</div>

---
<div class="content-section">
  <h3>📊 About the Dataset</h3>
  <p>
    The dataset includes car sales data such as car make, model, sale date, sale price,
    salesperson, and commission. This dashboard provides insights into brand
    pricing, top sales performers, and sales trends over time.
  </p>
</div>

---

```js
import { FileAttachment } from "observablehq:stdlib";
import * as Plot from "@observablehq/plot";
import * as d3 from "d3";
import * as Inputs from "@observablehq/inputs";

// Load CSV dataset
const data = await FileAttachment("./data/car_sales_data.csv").csv({ typed: true });
```
<style>
  .viz-grid {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
    justify-content: center;
    margin-bottom: 40px;
  }

  .viz-half {
    flex: 1 1 48%; /* Each half takes up ~48% of the row */
    max-width: 48%;
    box-sizing: border-box;
    padding: 15px;
    border: 2px solid #ccc;
    border-radius: 12px;
    background: #ffffff;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  }

  .viz-full {
    width: 100%;
    max-width: 1200px;
    margin: 20px auto;
    padding: 20px;
    border: 2px solid #ccc;
    border-radius: 12px;
    background: #ffffff;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  }
</style>

<div class="filter-grid"> <div class="filter-box">

## 📅 Filter by Year:
```js
// Dropdown: Year filter
const yearOptions = [...new Set(data.map(d => new Date(d["Date"]).getFullYear()))].sort();
const yearFilter = view(Inputs.select(yearOptions, {
  label: " Year:",
  value: yearOptions[0]
}));
```
</div> <div class="filter-box">

## 🚗 Filter by Car Model:
```js
// Dropdown: Car Model filter
const modelOptions = ["All", ...new Set(data.map(d => d["Car Model"]))].sort();
const modelFilter = view(Inputs.select(modelOptions, {
  label: " Car Model:",
  value: "All"
}));
```
</div>
</div>

```js
// Filter dataset based on selections
const filteredData = data.filter(d =>
  new Date(d["Date"]).getFullYear() === yearFilter &&
  (modelFilter === "All" || d["Car Model"] === modelFilter)
);
```
---
<div class="viz-grid"> 
<div class="viz-half">

## Sales trends 
```js
const makeModelAvg = d3.rollups(
  filteredData,
  v => d3.mean(v, d => d["Sale Price"]),
  d => d["Car Model"],
  d => d["Car Make"]
)
.flatMap(([model, group]) =>
  group.map(([make, avg]) => ({
    model,
    make,
    avg: +avg.toFixed(2)
  }))
);

// Plot heatmap
display(
  Plot.plot({
    width: 550,
    height: 400,
    marginLeft: 60,
    x: {
      label: "Car Make",
      domain: [...new Set(makeModelAvg.map(d => d.make))].sort()
    },
    y: {
      label: "Car Model",
      domain: [...new Set(makeModelAvg.map(d => d.model))].sort().reverse()
    },
    color: {
      label: "Avg Sale Price ($)",
      scheme: "greens",
      legend: true
    },
    marks: [
      Plot.cell(makeModelAvg, {
        x: "make",
        y: "model",
        fill: "avg",
        title: d => `${d.model} (${d.make})\nAvg Sale: $${d.avg.toLocaleString()}`
      }),
      Plot.text(makeModelAvg, {
        x: "make",
        y: "model",
        text: d => d.avg.toLocaleString(undefined, { maximumFractionDigits: 0 }),
        fill: "black",
        fontWeight: "bold"
      })
    ]
  })
)
```
</div>

<div class="viz-half">


## 🏆 Top 10 Salespersons by Total Commission

```js
const topCommissions = d3.rollups(
  filteredData,
  v => d3.sum(v, d => d["Commission Earned"]),
  d => d["Salesperson"]
).map(([name, total]) => ({ name, total }))
  .sort((a, b) => b.total - a.total)
  .slice(0, 10);

display(
  Plot.plot({
    width: 550,
    height: 400,
    marginLeft: 120,
    x: { label: "Total Commission Earned ($)" },
    y: {
      label: "Salesperson",
      domain: topCommissions.map(d => d.name).reverse()
    },
    marks: [
      Plot.barX(topCommissions, {
        x: "total",
        y: "name",
        fill: "#fb8500",
        tip: true,
        title: d => `${d.name}: $${d.total.toFixed(2)}`
      })
    ]
  })
)
```
</div>

</div>

---
<div class="viz-full">

<div>

## 💰 Avg Sale Price by Car Make

```js
const avgPriceByMake = d3.rollups(
  filteredData,
  v => d3.mean(v, d => d["Sale Price"]),
  d => d["Car Make"]
).map(([make, avg]) => ({ make, avg }));

display(
  Plot.plot({
    width: 800,
    height: 400,
    x: { label: "Car Make", domain: avgPriceByMake.map(d => d.make) },
    y: { label: "Avg Sale Price ($)" },
    marks: [
      Plot.barY(avgPriceByMake, {
        x: "make",
        y: "avg",
        fill: "#60a5fa",
        tip: true,
        title: d => `${d.make}: $${d.avg.toFixed(2)}`
      })
    ]
  })
)
```
</div>


</div>

<div class="content-section">
  <h3>📝 Final Insights and Observations</h3>
  <ul>
    <li>Car models like Corolla and Altima consistently maintain higher average sale prices across multiple car makes, indicating strong brand positioning and demand.</li>
    <li> Top-performing salespeople like Michael Smith and Michael Johnson contribute a significant portion of overall commissions, highlighting the importance of recognizing and retaining high-impact staff.</li>
    <li>Luxury brands like Honda and Toyota tend to command higher average sale prices, reflecting either premium offerings or successful upselling strategies.</li>
  </ul>
</div>

---