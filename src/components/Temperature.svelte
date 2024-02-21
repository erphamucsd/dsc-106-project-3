<script>
  import * as d3 from 'd3';

  // Assuming data is an array of arrays where each inner array represents data for a different year
  export let data;

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

  // Function to get the label position for the last point of each line
  function getLabelPosition(data) {
    const lastPoint = data[data.length - 1];
    return {
      x: x(new Date(2000, lastPoint.date.getUTCMonth(), lastPoint.date.getUTCDate())),
      y: y(lastPoint.value)
    };
  }

  function showTooltip(event, yearData) {
    const [year, value] = [yearData[0].date.getUTCFullYear(), yearData[yearData.length - 1].value]; // Example: year and the last value
    tooltip.innerHTML = `Year: ${year}<br>Value: ${value}`;
    tooltip.style.visibility = 'visible';
    tooltip.style.left = `${event.pageX + 15}px`;
    tooltip.style.top = `${event.pageY + 15}px`;
  }

  function hideTooltip() {
    tooltip.style.visibility = 'hidden';
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
        stroke-width="3"
        d={lineGenerator(yearData)} 
        on:mouseover={(event) => { hovered = i; }}
        on:mousemove={(event) => { hovered = i; }}
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
	>
		{#if hovered !== -1}
			{hovered + 1979}
		{/if}
	</div>

</div>