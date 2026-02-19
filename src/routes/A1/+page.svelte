<script lang="ts">
  import * as d3 from "d3";
  import { onMount } from "svelte";
  import type { TMovie } from "../../types";
  import Bar from "$lib/Bar.svelte";
  import BumpChart from "$lib/BumpChart.svelte";
  import Heatmap from "$lib/Heatmap.svelte";

  let movies: TMovie[] = [];

  async function loadCsv() {
    try {
      const csvUrl = "./summer_movies.csv";
      movies = await d3.csv(csvUrl, (row) => {
        return {
          ...row,
          num_votes: Number(row.num_votes),
          runtime_minutes: Number(row.runtime_minutes),
          average_rating: Number(row.average_rating),
          year: new Date(row.year ?? ""),
          genres: row.genres ? row.genres.split(",").map(g => g.trim()) : [],
        } as unknown as TMovie;
      });
      console.log("Loaded CSV Data:", movies);
    } catch (error) {
      console.error("Error loading CSV:", error);
    }
  }

  onMount(loadCsv);
</script>

<h1>Summer Movies</h1>
<p>Here are {movies.length == 0 ? "..." : movies.length + " "} movies</p>
<Bar {movies} />
<BumpChart {movies} />
<Heatmap {movies} />
