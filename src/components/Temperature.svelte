<script>
  import * as d3 from 'd3';

  export let data;

  const width = 928;
  const height = 600;
  const marginTop = 20;
  const marginRight = 30;
  const marginBottom = 30;
  const marginLeft = 40;

  let svg;

  // Placeholders for the axis elements.
  let gx;
  let gy;

  // Assuming data is an array of arrays, each inner array represents a dataset
  // Each inner array contains objects with 'date' and 'value' properties
  // Example: [[{ date: new Date(), value: 10 }, { date: new Date(), value: 15 }], [{ date: new Date(), value: 20 }, { date: new Date(), value: 25 }]]
  
  $: x = d3
    .scaleUtc()
    .domain([
      d3.min(data, dataset => d3.min(dataset, d => d.date)),
      d3.max(data, dataset => d3.max(dataset, d => d.date))
    ])
    .range([marginLeft, width - marginRight]);

  $: y = d3
    .scaleLinear()
    .domain([
      d3.min(data, dataset => d3.min(dataset, d => d.value)),
      d3.max(data, dataset => d3.max(dataset, d => d.value))
    ])
    .nice()
    .range([height - marginBottom, marginTop]);

  // Create a symmetric diverging color scale.
  $: max = d3.max(data.flat(), (d) => Math.abs(d.value));

  $: color = d3
    .scaleSequential()
    .domain([max, -max])
    .interpolator(d3.interpolateRdBu);

  $: d3.select(gx).call(d3.axisBottom(x).ticks(width / 80));
  $: d3.select(gy)
    .call(d3.axisLeft(y).ticks(null, '+'))
    // zero line
    .call((g) =>
      g
        .selectAll('.tick line')
        .clone()
        .attr('x2', width - marginRight - marginLeft)
        .attr('stroke-opacity', (d) => (d === 0 ? 1 : 0.1)),
    );

  const line = d3
    .line()
    .x((d) => x(d.date))
    .y((d) => y(d.value));

  const bisect = d3.bisector((d) => d.date).center;
  let tooltipPt = null;
  // function onPointerMove(event) {
  //   const i = bisect(data[0], x.invert(d3.pointer(event)[0])); // assuming all datasets have the same x values
  //   tooltipPt = data[0][i]; // assuming using the first dataset for tooltip
  // }
  function onPointerMove(event) {
    let closestDataPoint = null;
    let closestDistance = Infinity;

    data.forEach(dataset => {
      const i = bisect(dataset, x.invert(d3.pointer(event)[0]));
      const currentDataPoint = dataset[i];
      if (currentDataPoint) {
        const distance = Math.abs(x(currentDataPoint.date) - d3.pointer(event)[0]);
        if (distance < closestDistance) {
          closestDistance = distance;
          closestDataPoint = currentDataPoint;
        }
      }
    });

    tooltipPt = closestDataPoint;
  }
  function onPointerLeave(event) {
    tooltipPt = null;
  }

  $: d3.select(svg)
    .on('pointerenter pointermove', onPointerMove)
    .on('pointerleave', onPointerLeave);
</script>

<div class="temperature-plot">
  <svg
    bind:this={svg}
    {width}
    {height}
    viewBox="0 0 {width} {height}"
    style="max-width: 100%; height: auto;"
  >
    <!-- x-axis -->
    <g bind:this={gx} transform="translate(0,{height - marginBottom})" />
    <!-- y-axis -->
    <g bind:this={gy} transform="translate({marginLeft},0)">
      <text
        x="5"
        y={marginTop}
        dy="0.32em"
        fill="#000"
        font-weight="bold"
        text-anchor="start"
      >
        Anomaly (°C)
      </text>
    </g>

    <!-- lines -->
    {#each data as dataset, i}
      <path
        d={line(dataset)}
        fill="none"
        stroke={color(i)} 
        stroke-width="1.5"
      />
    {/each}

    <!-- tooltip -->
    {#if tooltipPt}
      <g transform="translate({x(tooltipPt.date)},{y(tooltipPt.value)})">
        <text font-weight="bold">{tooltipPt.value}</text>
      </g>
    {/if}
  </svg>
</div>
