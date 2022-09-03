<script>
  import { onMount } from "svelte";
  import { json, timeParse, select, scaleLinear, scaleTime } from "d3";
  import { extent } from "d3-array";
  import { line } from "d3-shape";
  import { axisLeft, axisBottom } from "d3-axis";

  const drawLineChart = async () => {
    const dataSet = await json("src/data/nyc_weather_data.json");
    const yAccessor = (d) => d.temperatureMax;
    const xAccessor = (d) => timeParse("%Y-%m-%d")(d.date);

    let dimensions = {
      width: window.innerWidth * 0.9,
      height: 400,
      margin: {
        top: 15,
        right: 15,
        bottom: 40,
        left: 60,
      },
    };

    dimensions.boundedWidth =
      dimensions.width - dimensions.margin.left - dimensions.margin.right;
    dimensions.boundedHeight =
      dimensions.height - dimensions.margin.bottom - dimensions.margin.top;

    const wrapper = select("#wrapper")
      .append("svg")
      .attr("width", dimensions.width)
      .attr("height", dimensions.height);

    const bounds = wrapper
      .append("g")
      .style(
        "transform",
        `translate(${dimensions.margin.left}px, ${dimensions.margin.top}px)`
      );

    const yScale = scaleLinear()
      .domain(extent(dataSet, yAccessor))
      .range([dimensions.boundedHeight, 0]);

    const xScale = scaleTime()
      .domain(extent(dataSet, xAccessor))
      .range([0, dimensions.boundedWidth]);

    const freezingTemperaturePlacement = yScale(32);

    const freezingTemperatures = bounds
      .append("rect")
      .attr("x", 0)
      .attr("width", dimensions.boundedWidth)
      .attr("y", freezingTemperaturePlacement)
      .attr("height", dimensions.boundedHeight - freezingTemperaturePlacement)
      .attr("fill", "#e0f3f3");

    const lineGenerator = line()
      .x((d) => xScale(xAccessor(d)))
      .y((d) => yScale(yAccessor(d)));

    bounds
      .append("path")
      .attr("d", lineGenerator(dataSet))
      .attr("fill", "none")
      .attr("stroke", "#af9358")
      .attr("stroke-width", 2);

    const yAxisGenerator = axisLeft().scale(yScale);

    const xAxisGenerator = axisBottom().scale(xScale);

    bounds.append("g").call(yAxisGenerator);

    bounds
      .append("g")
      .call(xAxisGenerator)
      .style("transform", `translateY(${dimensions.boundedHeight}px)`);
  };

  onMount(drawLineChart);
</script>

<div id="wrapper" />
