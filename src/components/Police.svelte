<script>
  import * as d3 from "d3";
  import { onMount } from "svelte";
  import { each } from "svelte/internal";

  let data = []; //store the csv for single plot
  let wholedata = []; // store whole data for allchart
  let inputCounty = "San Diego"; //store the input county name for the user
  let selectedData = []; //extract the county value from the whole csv
  let countyData = new Map();

  onMount(async () => {
    data = await d3.csv("./police_poplulation1.csv", (d) => ({
      county_name: d.county_name,
      data: Object.fromEntries(
        Object.entries(d)
          .filter(([key, _]) => key !== "county_name")
          .map(([key, value]) => [key, +value])
      ),
    }));

    console.log("all data", data);
    console.log("load csv success");

    // highlightChartData();  // Ensure initial data is loaded for default county

    allChart(data);
  });

  function allChart(dataArray) {
    dataArray.forEach(d => {
    countyData.set(d.county_name, Object.values(d.data));
    });
    // console.log("Processed county data", countyData);

    drawAllCounty();
  }

  function drawAllCounty() {
    //draw all county line
    console.log("Drawing all counties");

    const svg = d3.select("all_svg");
    const margin = { top: 20, right: 30, bottom: 30, left: 40 },
      width = 960 - margin.left - margin.right,
      height = 500 - margin.top - margin.bottom;

    const x = d3
      .scaleBand()
      .range([0, width])
      .domain(data.map((d) => d.yearlyData)) // Assuming 'year' is the property
      .padding(0.1);

    const y = d3
      .scaleLinear()
      .range([height, 0])
      .domain([0, d3.max(data, (d) => d.value)]); // Assuming 'value' is the property

    const g = svg
      .append("g")
      .attr("transform", `translate(${margin.left},${margin.top})`);

    g.append("g")
      .attr("class", "x axis")
      .attr("transform", `translate(0,${height})`)
      .call(d3.axisBottom(x));

    g.append("g").attr("class", "y axis").call(d3.axisLeft(y));

    g.selectAll(".bar")
      .data(data)
      .enter()
      .append("rect")
      .attr("class", "bar")
      .attr("x", (d) => x(d.year))
      .attr("width", x.bandwidth())
      .attr("y", (d) => y(d.value))
      .attr("height", (d) => height - y(d.value));
  }

  function highlightChartData() {
    const match = data.find((d) => d.county_name === inputCounty);
    if (match) {
      selectedData = Object.entries(match.data).map(([year, value]) => ({
        year,
        value,
      }));
      drawChart(); // Draw chart whenever data is updated
      //   console.log('Data for', inputCounty, selectedData);
    }
  }

  //draw the line chart for this county
  function drawChart() {
    const svg = d3.select("svg");
    svg.selectAll("*").remove(); // Clear previous drawings

    const margin = { top: 20, right: 100, bottom: 30, left: 50 },
      width = 800 - margin.left - margin.right,
      height = 500 - margin.top - margin.bottom;

    const x = d3
      .scaleBand()
      .domain(selectedData.map((d) => d.year))
      .range([0, width])
      .padding(0.1);

    const y = d3
      .scaleLinear()
      .domain([0, d3.max(selectedData, (d) => d.value)])
      .range([height, 0]);

    const g = svg
      .append("g")
      .attr("transform", `translate(${margin.left},${margin.top})`);

    g.append("g")
      .attr("transform", `translate(0,${height})`)
      .call(d3.axisBottom(x));

    g.append("g").call(d3.axisLeft(y));

    const line = d3
      .line()
      .x((d) => x(d.year) + x.bandwidth() / 2)
      .y((d) => y(d.value));

    g.append("path")
      .datum(selectedData)
      .attr("fill", "none")
      .attr("stroke", "steelblue")
      .attr("stroke-width", 2)
      .attr("d", line);
  }
</script>

<input type="text" bind:value={inputCounty} placeholder="Enter county name" />
<button on:click={highlightChartData}>Show Data</button>

<svg id="chart" width="750" height="500"></svg>

<p>Hello police data is below11</p>

<style>
  input {
    margin: 10px;
    padding: 8px;
    width: 200px;
  }
  button {
    padding: 8px 16px;
  }
  svg {
    margin-top: 20px;
    background-color: #f4f4f4;
    border: 1px solid #ddd;
  }
</style>
