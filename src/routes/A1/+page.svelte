<script lang="ts">
  import * as d3 from "d3";
  import { onMount } from "svelte";
  import type { TMovie } from "../../types";
  import Bar from "$lib/Bar.svelte";
  import Groupedbarchart from "$lib/Groupedbarchart.svelte";
  import Arcdiagram from "$lib/Arcdiagram.svelte";

  // Reactive variable for storing the data
  let movies: TMovie[] = $state([]);

  // Function to load the CSV
  async function loadCsv() {
    try {
      const csvUrl = "./summer_movies.csv";
      movies = await d3.csv(csvUrl, (row) => {
        // TIP: in row, all values are strings, so we need to use a row conversion function here to format them
        return {
          // ...row, // spread syntax to copy all properties from row
          // num_votes: Number(row.num_votes),
          // year: new Date(row.year),
          // please also format the values for other non-string attributes. You can check the attributes in the CSV file
            ...row,
            year: new Date(+row.year, 0, 1),
            runtime_minutes: +row.runtime_minutes,
            average_rating: +row.average_rating,
            num_votes: +row.num_votes,
            genres: row.genres ? row.genres.split(",") : [],
        };
      });

      console.log("Loaded CSV Data:", movies);
    } catch (error) {
      console.error("Error loading CSV:", error);
    }
  }
  // Call the loader when the component mounts
  onMount(loadCsv);
</script>

<h1>Summer Movies</h1>

<p>Here are {movies.length == 0 ? "..." : movies.length + " "} movies</p>
<Bar {movies} />
<Groupedbarchart {movies} />
<Arcdiagram {movies} />