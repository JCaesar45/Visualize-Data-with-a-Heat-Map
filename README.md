````markdown

# Heat Map Visualization Project

## Objective

The goal of this project is to create a heat map that displays global temperature data. The heat map should be functionally similar to [this example](https://heat-map.freecodecamp.rocks) and meet the listed user stories.

## Libraries/Technologies Used

- HTML
- CSS
- JavaScript
- D3.js (SVG-based visualization library)

## Data Source

- [Global Temperature Data](https://raw.githubusercontent.com/freeCodeCamp/ProjectReferenceData/master/global-temperature.json)

## Requirements

Your heat map visualization should meet the following user stories:

### User Stories

1. **Title**: The heat map should have a title with a corresponding `id="title"`.
2. **Description**: The heat map should have a description with a corresponding `id="description"`.
3. **X-axis**: The heat map should have an x-axis with a corresponding `id="x-axis"`.
4. **Y-axis**: The heat map should have a y-axis with a corresponding `id="y-axis"`.
5. **Cells**: The heat map should have `rect` elements with a class `cell` that represent the data.
6. **Cell Colors**: There should be at least 4 different fill colors used for the cells.
7. **Cell Data**: Each cell will have the properties `data-month`, `data-year`, `data-temp` containing their corresponding month, year, and temperature values.
8. **Data Range**: The `data-month`, `data-year` of each cell should be within the range of the data.
9. **Cell Alignment (Y-axis)**: Cells should align with the corresponding month on the y-axis.
10. **Cell Alignment (X-axis)**: Cells should align with the corresponding year on the x-axis.
11. **Month Labels (Y-axis)**: The y-axis should have multiple tick labels with the full month name.
12. **Year Labels (X-axis)**: The x-axis should have multiple tick labels with the years between 1754 and 2015.
13. **Legend**: The heat map should have a legend with a corresponding `id="legend"`.
14. **Legend Rectangles**: The legend should contain `rect` elements.
15. **Legend Colors**: The `rect` elements in the legend should use at least 4 different fill colors.
16. **Tooltip**: When the user mouses over a cell, a tooltip should appear with a corresponding `id="tooltip"` displaying more information about the area.
17. **Tooltip Data**: The tooltip should have a `data-year` property that corresponds to the `data-year` of the active area.

## Code Example

Here is an example of how the HTML, CSS, and JavaScript might be structured:

### HTML

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Global Temperature Heat Map</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <div id="title">Global Temperature Heat Map</div>
  <div id="description">
    This heat map visualizes global temperature data over time from 1754 to 2015.
  </div>
  <svg width="100%" height="500" id="heat-map"></svg>
  <div id="legend"></div>
  <div id="tooltip" class="tooltip"></div>

  <script src="https://d3js.org/d3.v5.min.js"></script>
  <script src="app.js"></script>
</body>
</html>
```

### CSS

```css
#heat-map {
  width: 90%;
  margin: 20px auto;
  display: block;
}

#tooltip {
  position: absolute;
  visibility: hidden;
  background: rgba(0, 0, 0, 0.7);
  color: #fff;
  padding: 10px;
  border-radius: 5px;
  font-size: 12px;
}

#legend {
  display: flex;
  justify-content: space-between;
  margin: 20px auto;
  width: 90%;
}

.cell {
  cursor: pointer;
}

.cell:hover {
  opacity: 0.7;
}
```

### JavaScript

```javascript
// Load the data
d3.json('https://raw.githubusercontent.com/freeCodeCamp/ProjectReferenceData/master/global-temperature.json').then(data => {
  const margin = { top: 50, right: 30, bottom: 50, left: 40 };
  const width = 1000 - margin.left - margin.right;
  const height = 400 - margin.top - margin.bottom;

  const svg = d3.select('#heat-map')
    .attr('width', width + margin.left + margin.right)
    .attr('height', height + margin.top + margin.bottom)
    .append('g')
    .attr('transform', `translate(${margin.left},${margin.top})`);

  const xScale = d3.scaleBand()
    .domain(data.monthlyVariance.map(d => d.year))
    .range([0, width])
    .padding(0.1);

  const yScale = d3.scaleBand()
    .domain(data.monthlyVariance.map(d => d.month))
    .range([0, height])
    .padding(0.1);

  const colorScale = d3.scaleSequential(d3.interpolateWarm)
    .domain([d3.min(data.monthlyVariance, d => d.variance), d3.max(data.monthlyVariance, d => d.variance)]);

  // Create x and y axes
  svg.append('g')
    .attr('id', 'x-axis')
    .attr('transform', `translate(0,${height})`)
    .call(d3.axisBottom(xScale).tickFormat(d3.format('d')));

  svg.append('g')
    .attr('id', 'y-axis')
    .call(d3.axisLeft(yScale).tickFormat(month => d3.timeFormat('%B')(new Date(0, month))));

  // Add the cells
  svg.selectAll('.cell')
    .data(data.monthlyVariance)
    .enter()
    .append('rect')
    .attr('class', 'cell')
    .attr('x', d => xScale(d.year))
    .attr('y', d => yScale(d.month))
    .attr('width', xScale.bandwidth())
    .attr('height', yScale.bandwidth())
    .attr('data-month', d => d.month)
    .attr('data-year', d => d.year)
    .attr('data-temp', d => data.baseTemperature + d.variance)
    .style('fill', d => colorScale(data.baseTemperature + d.variance))
    .on('mouseover', function(event, d) {
      const tooltip = d3.select('#tooltip');
      tooltip.style('visibility', 'visible')
        .attr('data-year', d.year)
        .html(`${d3.timeFormat('%B')(new Date(0, d.month))} ${d.year} - ${Math.round(data.baseTemperature + d.variance)}°C`);
    })
    .on('mouseout', function() {
      d3.select('#tooltip').style('visibility', 'hidden');
    });

  // Create the legend
  const legend = d3.select('#legend');
  const legendWidth = 300;
  const legendHeight = 20;

  const legendScale = d3.scaleLinear()
    .domain([d3.min(data.monthlyVariance, d => d.variance), d3.max(data.monthlyVariance, d => d.variance)])
    .range([0, legendWidth]);

  const legendAxis = d3.axisBottom(legendScale)
    .ticks(4)
    .tickFormat(d => Math.round(data.baseTemperature + d));

  legend.append('svg')
    .attr('width', legendWidth)
    .attr('height', legendHeight)
    .selectAll('rect')
    .data(colorScale.range())
    .enter()
    .append('rect')
    .attr('x', (d, i) => i * (legendWidth / 4))
    .attr('width', legendWidth / 4)
    .attr('height', legendHeight)
    .style('fill', d => d);

  legend.append('g')
    .attr('transform', `translate(0, ${legendHeight})`)
    .call(legendAxis);
});
```

### Steps to Run

1. Clone this repository or create a new CodePen.
2. Copy the HTML, CSS, and JavaScript into their respective files.
3. Open `index.html` in a browser to view the heat map in action.
4. Ensure all tests pass before submitting the link to your project.
