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

  // Sorts csv file by year into an array of arrays, with each inner array
  // representing a dataset for the year and containing date/temp objects
  function date_sort(parsed_csv) {
    const data = parsed_csv.reduce((acc, item) => {
      const date = new Date(item.date);
      const year = date.getFullYear();
      if (!acc[year]) {
        acc[year] = []; // Initialize array for each year if not already done
      }
      acc[year].push({'date': date, 'value': Number(item.sst)});
      return acc;
    }, []);
    return Object.values(data); // Return the sorted data as an array of arrays
}

  data = date_sort(parsed)
  console.log(data); // Output the sorted data   
 
});

  function update(selectedGroup) {

        // Create new data with the selection?
        var dataFilter = data.filter(function(d){return d.name==selectedGroup})

        // Give these new data to update line
        line
            .datum(dataFilter)
            .transition()
            .duration(1000)
            .attr("d", d3.line()
              .x(function(d) { return x(d.year) })
              .y(function(d) { return y(+d.n) })
            )
            .attr("stroke", function(d){ return myColor(selectedGroup) })
      }

      // When the button is changed, run the updateChart function
      d3.select("#selectButton").on("change", function(d) {
          // recover the option that has been chosen
          var selectedOption = d3.select(this).property("value")
          // run the updateChart function with this selected option
          update(selectedOption)
      })

</script>

<main>
  <h1>Global Sea Surface Temperature Trends</h1>
  <Temperature {data} />
  <input placeholder="Year" value="" />
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

  input {
    top: 200;
    left: 200;
  }
</style>
