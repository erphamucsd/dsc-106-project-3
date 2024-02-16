<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import Temperature from './Temperature.svelte';
 
  let data = [];

  onMount(async () => {
  const res = await fetch(
    'https://raw.githubusercontent.com/erphamucsd/dsc-106-project-3/main/assets/Sea_Surface_Temp_Cleaned.csv',
  );
  const csv = await res.text();

  var parsed = d3.csvParse(csv);

  // sorts data from parsed into an array of arrays, 
  // with each array being a dataset for a year
  function date_sort(parsed) { 
    parsed.forEach(item => {
      const date = new Date(item.date);
      const year = date.getFullYear(); // Use getFullYear() instead of getYear()
      if (!data[year]) {
        data[year] = []; // Initialize array for each year if not already done
      }
      data[year].push({'date':date, 'value': Number(item.sst)});
    });
    return data; // Return the sorted data
  }
  data = date_sort(parsed)
  console.log(date_sort(parsed)); // Output the sorted data   

  });
</script>

<main>
  <h1>Global Sea Surface Temperature Trends</h1>
  <Temperature {data} />
</main>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Nunito:wght@300;400;700&display=swap');

  :root {
    --color-bg: #ffffff;
    --color-outline: #c2c2c2;
    --color-shadow: hsl(0, 0%, 0%, 0.1);
    --color-text: #3f4252;
    --color-bg-1: hsla(0, 0%, 0%, 0.2);
    --color-shadow-1: hsl(0, 0%, 96%);
  }

  *,
  *::before,
  *::after {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  main {
    text-align: center;
    font-family: 'Nunito', sans-serif;
    font-weight: 300;
    line-height: 2;
    font-size: 24px;
    color: var(--color-text);
  }

  h1 {
    font-size: 2em;
    font-weight: 300;
    line-height: 2;
  }
</style>

