<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import Temperature from './Temperature.svelte';
 
  let data = [];
  let data_dict = [];
  let start_year_text = "";
  let end_year_text = "";

  onMount(async () => {
  const res = await fetch(
    'https://raw.githubusercontent.com/erphamucsd/dsc-106-project-3/main/assets/Sea_Surface_Temp_Cleaned.csv',
  );
  const csv = await res.text();
  var parsed = d3.csvParse(csv);

  // Sorts csv file by year into an array of arrays, with each inner array
  // representing a dataset for the year and containing date/temp objects
  function date_sort(parsed_csv) {
    const data = parsed_csv.reduce((acc, item) => {
      const date = new Date(item.date);
      const year = date.getUTCFullYear();
      if (!acc[year]) {
        acc[year] = []; // Initialize array for each year if not already done
      }
      acc[year].push({'date': date, 'value': Number(item.sst)});
      return acc;
    }, []);
    return data; // Return the sorted data as an array of arrays
  }
  
  data_dict = date_sort(parsed)
  data = Object.values(date_sort(parsed))

  });

  function filter_data(data_dict, start, end) {
    let filtered_data = [];
    for (let key in data_dict) {
      if (end == "") {
        if (parseInt(key) >= start) {
          filtered_data[key] = data_dict[key];
        }
      } else {
        if (parseInt(key) >= start && parseInt(key) <= end) {
          filtered_data[key] = data_dict[key];
        }
      }
    }
    data = Object.values(filtered_data)
    return data;
  }

  // testing some things
  // filtered_data = Object.values(filter_data(date_sort(parsed), 1990, 2025))
  // console.log(filtered_data)
  // console.log(Object.entries(filtered_data))
  // console.log(Object.entries(data))
  //console.log(decades)

</script>

<main>
  <h1>Global Sea Surface Temperature Trends</h1>
  <Temperature {data} />
  <input 
    placeholder="Start Year" 
    bind:value={start_year_text} />
  <input 
    placeholder="End Year"
    bind:value={end_year_text} />
  <button on:click={filter_data(data_dict, start_year_text, end_year_text)}>
  Filter
  </button>
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
    font-size: 1.5em;
    font-weight: 300;
    line-height: 2;
  }

  input {
    top: 200;
    left: 200;
  }
</style>
