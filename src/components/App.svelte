<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import Temperature from './Temperature.svelte';
 
  let data = [];
  let data_dict = [];
  let start_year_text = "1979";
  let end_year_text = "2024";

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
</script>

<main>
  <h1>Record-Breaking Heat is Breaking our Planet</h1>
  <h3>Just one of many signs of a rapidly changing planet, sea surface temperature data collected since 1979 indicates that our oceans are experiencing record warming.</h3>
  <div class="container">
    <div class="wrapper">
      <Temperature {data} {start_year_text} />
    </div>
    <div class="content">
      <h6>Filter by year here:</h6>
      <input 
        placeholder="Start Year (1979)" 
        bind:value={start_year_text} />
      <input 
        placeholder="End Year (2024)"
        bind:value={end_year_text} />
      <button on:click={filter_data(data_dict, start_year_text, end_year_text)}>
      Filter
      </button>
    </div>
  </div>
  <h2>Write-Up:</h2>
  <div class="writeup1">
    <p> For our project, we decided to make a visualization depicting the change in the sea surface temperature over the months. Since we’re tracking the change in temperature over time, we utilized a line chart to create our graph. We encoded a line for every year, starting from 1979 to 2024. To differentiate the lines, we represented each year with a divergent color scheme, with earlier years corresponding to a more saturated blue color and later years corresponding to a saturated red color to reinforce the redundancy of the comparisons in the visual encodings between each year. Lastly, we scaled the y-axis to start at 19.6 degrees rather than 0 degrees because all of the temperatures are above 19.6 degrees. By doing this, we can zoom in on the visualization and have a better view of the trends.
    </p>
    <p> Since our visualization contains data for so many years, it makes it difficult to discern the pattern for a particular year. Due to this issue, we decided to make our interaction filter out the specific years that the reader wants to see. Our interaction allows the reader to type in the starting year and end year that they want to look up. Once the reader presses the “Filter” button, the graph displays the lines that correspond to the range of years that they selected. Furthermore, we added an interaction element that allows the reader to see what year each line corresponds to if the reader hovers over a specific line. Utilizing the hover tooltip helps declutter the visualization so that we don’t have to create a long legend for all of the years.
    </p>
    <p> There were many possible interactions that we were considering, with one such option being a dropdown menu where the reader can select a specific year to view rather than input the year into a search box. However, we decided not to use this interaction technique because of the large amount of years that we have in our data. Since we have a total of 45 different years in our data, the dropdown menu would have to contain 45 different values, which would make it very long and cumbersome to interact with. Another possible interaction choice we were thinking about was making the nonfiltered lines gray while the filtered lines remained colored. By doing this, we were hoping that the reader would be able to contrast the trends between their selected years with the rest of the years in the visualization. However, we realized that doing this would make our visualization cluttered. Therefore, we decided it would be best to only display the filtered years in our graph.
    </p>
  </div>
  <div class="writeup2">
    <p> 	We chose to split our project’s tasks into two main parts: displaying the line chart and developing the interaction elements. Katelyn created the line chart by first figuring out how to query the data into an array of arrays, with each array corresponding to one year. She then used a for loop to go through the array and plot a line for each year. Katelyn also created the graph and made the axes titles. All of this took approximately 8 hours.
    </p>
    <p> Eric and Vanessa were in charge of making the interaction aspect of the visualization. Eric created a function that filters the data to display only the years that the reader searched for. Vanessa made the input boxes and the “Filter” button, which she then connected to the function that Eric made once the button was clicked on. They both worked on creating the hover tooltip that displays the year for each line, with Eric figuring out the final code for it. 
    </p>
    <p> The interaction section was the most time-consuming and challenging part of the project. It was difficult to figure out what functions we needed and how to modify our existing functions to connect all of the different elements. It took us a total of about 18 hours of tinkering with the code and going to office hours before we finished this part of the project.
    </p>
  </div>
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
    font-size: 1.75em;
    font-weight: 500;
    line-height: 2;
  }

  h2 {
    font-size: 1em;
    font-weight: 400;
    line-height: 2;
    text-align: left;
    text-decoration: underline;
  }

  h3 {
    font-size: 0.8em;
    font-weight: 400;
    line-height: 2;
  }

  h6 {
    font-size: 0.8em;
    font-weight: 400;
  }

  p {
    font-size: 0.7em;
    font-weight: 300;
    line-height: 1.75;
    text-align: left;
    text-indent: 40px;
  }
  
  .writeup1{
    margin-top: 5px;
    margin-left: 10px;
    margin-right: 10px;
  }

  .writeup2{
    margin-top: 17px;
    margin-left: 10px;
    margin-right: 10px;
  }
  .container{
    display: flex;
    justify-content: center;
  }

  .content{
    display: flex;
    flex-direction: column;
    padding: 100px;
    gap: 10px;
    justify-content: center;
  }

  input {
    top: 200;
    left: 200;
    font-family: 'Nunito', sans-serif;
    width: 150px;
    margin-left: 10px;
  }

  button {
    font-family: 'Nunito', sans-serif;
    width: 50px;
    text-align: center;
    margin-left: 60px;
    margin-top: 5px;
  }
</style>
