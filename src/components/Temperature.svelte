<script>
  import * as d3 from 'd3';

  // Assuming data is an array of arrays where each inner array represents data for a different year
  export let data;
  export let start_year_text;

  const width = 800;
  const height = 440;
  const marginTop = 20;
  const marginRight = 30;
  const marginBottom = 35;
  const marginLeft = 58;

  let hovered = -1;
  let recorded_mouse_position = {
		x: 0, y: 0
	};

  let gx;
  let gy;

  $: start = parseInt(start_year_text)
  
  // Domain spans 2000 only so that we can use a base year to place all the lines
  $: x = d3
      .scaleTime()
      .domain([
          new Date(Date.UTC(2000, 0, 1)), // January 1, 2000
          new Date(Date.UTC(2000, 11, 31)) // December 31, 2000
      ])
      .range([marginLeft, width - marginRight]);

  // Uses flatmap, which turns the nested arrays into one array and finds min/max
  $: y = d3
      .scaleLinear()
      .domain([
          d3.min(data.flatMap(d => d.map(item => item.value))) - 0.1,
          d3.max(data.flatMap(d => d.map(item => item.value))) + 0.1
      ])
      .range([height - marginBottom, marginTop]);

  // Generates the line for each year's dataset using the month, date, and temp value
  $: lineGenerator = d3.line()
      .x(d => x(new Date(2000, d.date.getUTCMonth(), d.date.getUTCDate()))) // Using base year 2000 to ensure consistent plotting
      .y(d => y(d.value));

  // Creates a color scale for all the years from 1978-2024, with
  // red representing later years and blue representing earlier years
  $: color = d3.scaleSequential()
      .domain([0, data.length - 1]) // Domain from 0 to the number of years - 1
      .interpolator(t => d3.interpolateRdYlBu(1 - t)); // Interpolate from blue to red

  // Create the axes
  $: {
    if (gx) d3.select(gx).call(d3.axisBottom(x).tickFormat(d3.timeFormat("%b"))); // Set the tick format to display only the month abbreviation
    if (gy) d3.select(gy).call(d3.axisLeft(y));
  }
</script>

<div class="temperature-plot">
  <svg
    {width}
    {height}
    viewBox={`0 0 ${width} ${height}`}
    style="max-width: 100%; height: auto;"
  >
    <!-- Add X axis -->
    <g bind:this={gx} transform={`translate(0, ${height - marginBottom})`} />
    <!-- Add Y axis -->
    <g bind:this={gy} transform={`translate(${marginLeft}, 0)`} />

    <!-- Loop through each set of data for a different year -->
    {#each data as yearData, i}
      <path
        key={i}
        fill="none"
        stroke={color(i)}
        stroke-width="3.6"
        d={lineGenerator(yearData)} 
        on:mouseover={(event) => { 
          hovered = i; 
          recorded_mouse_position = {
							x: event.pageX,
							y: event.pageY
						}
        }}
        on:mouseout={(event) => { hovered = -1; }}
      />
    {/each}

    <!-- Y-axis label -->
    <text  
      transform={`translate(${14},${2*height  /  3}) rotate(-90)`}
      font-size=14
      >Temperature (Celsius)</text>

    <!-- X-axis label -->
    <text  
      transform={`translate(${width/2},${height})`}
      font-size=14
      >Month</text>
  </svg>
  <div
		class={hovered === -1 ? "tooltip-hidden": "tooltip-visible"}	
    style="left: {recorded_mouse_position.x + 10}px; top: {recorded_mouse_position.y}px"
	>
		{#if hovered !== -1}
			{hovered + start}
		{/if}
	</div>

</div>

<style>
	/* dynamic classes for the tooltip */
	.tooltip-hidden {
		visibility: hidden;
		font-family: "Nunito", sans-serif;
		width: 200px;
		position: absolute;
	}

	.tooltip-visible {
		font: 18px sans-serif;
		font-family: "Nunito", sans-serif;
		visibility: visible;
		background-color: #F5F5F4;
		border-radius: 10px;
		width: 50px;
		color: black;
		position: absolute;
		padding: 10px;
    border: 2px solid #2A2826;

	}
</style>
