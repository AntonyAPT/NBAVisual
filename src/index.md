# NBA Shot Chart

2019–20 NBA Season

```js
import * as d3 from "npm:d3";
```

```js
const shots = await FileAttachment("data/shots.json").json();
```

```js
const players = [...new Set(shots.map((d) => d["Player Name"]))];
```

```js
const selectedPlayer = view(
  Inputs.select(players, { label: "Select a player" }),
);
```

```js
const selectedResult = view(
  Inputs.radio(["All", "Made", "Missed"], {
    label: "Shot result",
    value: "All",
  }),
);
```

```js
const playerShots = shots.filter((d) => {
  const playerMatch = d["Player Name"] === selectedPlayer;
  const resultMatch =
    selectedResult === "All"
      ? true
      : selectedResult === "Made"
        ? d["Shot Made Flag"] === 1
        : d["Shot Made Flag"] === 0;
  return playerMatch && resultMatch;
});
```

```js
function drawCourt(container) {
  const width = 500;
  const height = 470;

  const svg = d3
    .select(container)
    .append("svg")
    .attr("width", width)
    .attr("height", height);

  const court = svg.append("g").attr("transform", "translate(250, 50)");

  // Court background
  court
    .append("rect")
    .attr("x", -250)
    .attr("y", 0)
    .attr("width", 500)
    .attr("height", 470)
    .attr("fill", "#f5deb3")
    .attr("stroke", "#000")
    .attr("stroke-width", 1);

  // Paint / key
  court
    .append("rect")
    .attr("x", -80)
    .attr("y", 0)
    .attr("width", 160)
    .attr("height", 190)
    .attr("fill", "none")
    .attr("stroke", "#000")
    .attr("stroke-width", 1);

  // Free throw circle
  court
    .append("circle")
    .attr("cx", 0)
    .attr("cy", 190)
    .attr("r", 60)
    .attr("fill", "none")
    .attr("stroke", "#000")
    .attr("stroke-width", 1);

  // Backboard
  court
    .append("line")
    .attr("x1", -30)
    .attr("y1", 40)
    .attr("x2", 30)
    .attr("y2", 40)
    .attr("stroke", "#000")
    .attr("stroke-width", 2);

  // Basket support line
  court
    .append("line")
    .attr("x1", 0)
    .attr("y1", 40)
    .attr("x2", 0)
    .attr("y2", 52)
    .attr("stroke", "#000")
    .attr("stroke-width", 1);

  // Basket
  court
    .append("circle")
    .attr("cx", 0)
    .attr("cy", 60)
    .attr("r", 7.5)
    .attr("fill", "none")
    .attr("stroke", "#000")
    .attr("stroke-width", 1);

  // Restricted area arc
  court
    .append("path")
    .attr("d", "M -40 60 A 40 40 0 0 0 40 60")
    .attr("fill", "none")
    .attr("stroke", "#000")
    .attr("stroke-width", 1);

  // Three point corner lines
  court
    .append("line")
    .attr("x1", -220)
    .attr("y1", 0)
    .attr("x2", -220)
    .attr("y2", 140)
    .attr("stroke", "#000")
    .attr("stroke-width", 1);

  court
    .append("line")
    .attr("x1", 220)
    .attr("y1", 0)
    .attr("x2", 220)
    .attr("y2", 140)
    .attr("stroke", "#000")
    .attr("stroke-width", 1);

  // Three point arc
  court
    .append("path")
    .attr("d", "M -220 140 A 237.5 237.5 0 0 0 220 140")
    .attr("fill", "none")
    .attr("stroke", "#000")
    .attr("stroke-width", 1);

  return svg;
}
```

```js
const tooltip = d3
  .select("body")
  .append("div")
  .style("position", "absolute")
  .style("background", "rgba(0,0,0,0.75)")
  .style("color", "white")
  .style("padding", "8px 12px")
  .style("border-radius", "6px")
  .style("font-size", "12px")
  .style("pointer-events", "none")
  .style("opacity", 0);
```

```js
function plotShots(svg, shots) {
  const xScale = d3.scaleLinear().domain([-250, 250]).range([0, 500]);

  const yScale = d3.scaleLinear().domain([-50, 420]).range([0, 470]);

  const validShots = shots.filter(
    (d) =>
      d["X Location"] >= -250 &&
      d["X Location"] <= 250 &&
      d["Y Location"] >= -50 &&
      d["Y Location"] <= 420,
  );

  svg.selectAll("circle.shot").remove();

  svg
    .selectAll("circle.shot")
    .data(validShots)
    .join("circle")
    .attr("class", "shot")
    .attr("cx", (d) => xScale(d["X Location"]))
    .attr("cy", (d) => yScale(d["Y Location"]) + 50)
    .attr("r", 4)
    .attr("fill", (d) => (d["Shot Made Flag"] === 1 ? "#378ADD" : "#E24B4A"))
    .attr("opacity", 0.6)
    .on("mouseover", function (event, d) {
      tooltip.style("opacity", 1).html(`
          <strong>${d["Player Name"]}</strong><br/>
          ${d["Shot Made Flag"] === 1 ? "✓ Made" : "✗ Missed"}<br/>
          ${d["Action Type"]}<br/>
          ${d["Shot Distance"]} ft<br/>
        `);
    })
    .on("mousemove", function (event) {
      tooltip
        .style("left", event.pageX + 12 + "px")
        .style("top", event.pageY - 28 + "px");
    })
    .on("mouseout", function () {
      tooltip.style("opacity", 0);
    });
}
```

```js
const wrapper = display(html`
  <div style="display:flex; align-items:flex-start; gap:16px;">
    <div id="court"></div>
    <div
      style="display:flex; flex-direction:column; gap:10px; font-size:13px; padding-top:50px;"
    >
      <span
        ><svg width="12" height="12">
          <circle cx="6" cy="6" r="5" fill="#378ADD" opacity="0.8" />
        </svg>
        Made</span
      >
      <span
        ><svg width="12" height="12">
          <circle cx="6" cy="6" r="5" fill="#E24B4A" opacity="0.8" />
        </svg>
        Missed</span
      >
    </div>
  </div>
`);
const svg = drawCourt(wrapper.querySelector("#court"));
plotShots(svg, playerShots);
```
