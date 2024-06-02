<script>
  // data is a list of objects, each with the following keys:
  // { name, sex, year, count, rank }
  import { onMount } from "svelte";
  // export let data;

  import * as d3 from "d3";

  let data = [];
  let countyName = ''; // You can set this dynamically based on user input

  // Load data from CSV
  onMount(async () => {
    data = await d3.csv('./temp_all_pop.csv');
    // Transpose the data
    let transposedData = d3.transpose(data.map(d => Object.values(d)));
    let years = transposedData[0].slice(1);
    let counties = data.columns.slice(1);

    data = years.map((year, i) => {
      let obj = { Year: year };
      counties.forEach((county, j) => {
        obj[county] = +transposedData[j + 1][i + 1];
      });
      return obj;
    });

    drawChart();
  });

  function drawChart() {
    const svg = d3.select("svg");
    svg.selectAll("*").remove();

    const margin = { top: 20, right: 30, bottom: 30, left: 40 },
          width = +svg.attr("width") - margin.left - margin.right,
          height = +svg.attr("height") - margin.top - margin.bottom,
          g = svg.append("g").attr("transform", `translate(${margin.left},${margin.top})`);

    const x = d3.scaleLinear()
        .domain(d3.extent(data, d => +d.Year))
        .range([0, width]);

    const y = d3.scaleLinear()
        .domain([0, d3.max(data, d => d3.max(Object.values(d).slice(1)))])
        .nice()
        .range([height, 0]);

    const line = d3.line()
        .x(d => x(+d.Year))
        .y(d => y(d.value));

    g.append("g")
        .attr("transform", `translate(0,${height})`)
        .call(d3.axisBottom(x));

    g.append("g")
        .call(d3.axisLeft(y));

    if (countyName && data.columns.includes(countyName)) {
      const countyData = data.map(d => ({ Year: +d.Year, value: +d[countyName] }));
      g.append("path")
        .datum(countyData)
        .attr("fill", "none")
        .attr("stroke", "steelblue")
        .attr("stroke-width", 1.5)
        .attr("d", line);
    } else {
      data.columns.slice(1).forEach(county => {
        const countyData = data.map(d => ({ Year: +d.Year, value: +d[county] }));
        g.append("path")
          .datum(countyData)
          .attr("fill", "none")
          .attr("stroke", () => d3.schemeCategory10[Math.floor(Math.random() * 10)])
          .attr("stroke-width", 1.5)
          .attr("d", line);
      });
    }
  }
</script>

<style>
  svg {
    font: 10px sans-serif;
  }
  .axis path,
  .axis line {
    fill: none;
    shape-rendering: crispEdges;
  }
</style>

<svg width="960" height="500"></svg>
