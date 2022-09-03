<script>
  import {
    axisBottom,
    axisLeft,
    extent,
    json,
    min,
    scaleLinear,
    select,
  } from "d3";
  import { onMount } from "svelte";

  const xAccessor = (d) => d.dewPoint;
  const yAccessor = (d) => d.humidity;
  const colorAccessor = (d) => d.cloudCover;

  const width = min([window.innerWidth, window.innerHeight]) * 0.9;

  let dimensions = {
    width,
    height: width,
    margin: {
      top: 10,
      right: 10,
      bottom: 50,
      left: 50,
    },
  };

  dimensions.boundedHeight =
    dimensions.height - dimensions.margin.top - dimensions.margin.bottom;
  dimensions.boundedWidth =
    dimensions.width - dimensions.margin.left - dimensions.margin.right;

  const drawChart = async () => {
    const dataSet = await json("src/data/nyc_weather_data.json");

    const wrapper = select("#wrapper")
      .append("svg")
      .attr("height", dimensions.height)
      .attr("width", dimensions.width);

    const bounds = wrapper
      .append("g")
      .attr("height", dimensions.boundedHeight)
      .attr("width", dimensions.boundedWidth)
      .style(
        "transform",
        `translate(${dimensions.margin.left}px, ${dimensions.margin.top}px)`
      );

    const xScale = scaleLinear()
      .domain(extent(dataSet, xAccessor))
      .range([0, dimensions.boundedWidth])
      .nice();

    const yScale = scaleLinear()
      .domain(extent(dataSet, yAccessor))
      .range([dimensions.boundedHeight, 0])
      .nice();

    const colorScale = scaleLinear()
      .domain(extent(dataSet, colorAccessor))
      .range(["skyblue", "darkslategrey"]);

    bounds
      .selectAll("circle")
      .data(dataSet)
      .enter()
      .append("circle")
      .attr("cx", (d) => xScale(xAccessor(d)))
      .attr("cy", (d) => yScale(yAccessor(d)))
      .attr("r", 5)
      .attr("fill", (d) => colorScale(colorAccessor(d)));

    const xAxisGenerator = axisBottom().scale(xScale);

    const xAxis = bounds
      .append("g")
      .call(xAxisGenerator)
      .style("transform", `translateY(${dimensions.boundedHeight}px)`);

    xAxis
      .append("text")
      .attr("fill", "black")
      .attr("x", dimensions.boundedWidth / 2)
      .attr("y", dimensions.margin.bottom - 10)
      .style("font-size", "1.4em")
      .html("Dew point (&deg;F)");

    const yAxisGenerator = axisLeft().scale(yScale).ticks(4);

    const yAxis = bounds.append("g").call(yAxisGenerator);

    yAxis
      .append("text")
      .attr("x", -dimensions.boundedHeight / 2)
      .attr("y", -dimensions.margin.left + 10)
      .attr("fill", "black")
      .style("font-size", "1.4em")
      .style("transform", "rotate(-90deg)")
      .style("text-anchor", "middle")
      .html("Relative humidity");
  };

  onMount(drawChart);
</script>

<div id="wrapper" />
