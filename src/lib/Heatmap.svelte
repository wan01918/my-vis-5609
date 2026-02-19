<script lang="ts">
  import type { TMovie } from "../types";
  import * as d3 from "d3";

  type Props = {
    movies: TMovie[];
    width?: number;
    height?: number;
  };

  let { movies, width = 1000, height = 1000 }: Props = $props();
  let hoveredCell: any = $state(null);

  const margin = {
    top: 100,
    bottom: 30,
    left: 100,
    right: 30,
  };

  function getCoOccurrence(movies: TMovie[]) {
    let coCount: { [key: string]: number } = {};
    let allGenres = new Set();

    movies.forEach((movie) => {
      let genres = movie.genres;
      for (let k = 0; k < genres.length; k++) {
        allGenres.add(genres[k]);
      }
      for (let i = 0; i < genres.length; i++) {
        for (let j = 0; j < genres.length; j++) {
          if (i !== j) {
            let key = genres[i] + "-" + genres[j];
            coCount[key] = (coCount[key] || 0) + 1;
          }
        }
      }
    });

    let genres = Array.from(allGenres).sort();
    let matrix = [];

    for (let i = 0; i < genres.length; i++) {
      for (let j = 0; j < genres.length; j++) {
        let key = genres[i] + "-" + genres[j];
        matrix.push({ g1: genres[i], g2: genres[j], count: coCount[key] || 0 });
      }
    }

    return { genres, matrix };
  }

  const coData = $derived(getCoOccurrence(movies));

  const xScale = $derived(
    d3.scaleBand()
      .domain(coData.genres)
      .range([margin.left, width - margin.right])
  );

  const yScale = $derived(
    d3.scaleBand()
      .domain(coData.genres)
      .range([margin.top, height - margin.bottom])
  );

  const maxCount = $derived(Math.max(...coData.matrix.map((d) => d.count)));

  const colorScale = $derived(
    d3.scaleSequential(d3.interpolateYlOrRd).domain([0, maxCount])
  );

  const xCellWidth: number = $derived(xScale.bandwidth());
  const yCellHeight: number = $derived(yScale.bandwidth());

</script>

<h3>Q2: Genre Co-occurrence Matrix</h3>

{#if movies.length > 0}
  <svg {width} {height}>
     {#each coData.matrix as cell}
      <rect
        x={xScale(cell.g1)}
        y={yScale(cell.g2)}
        width={xCellWidth}
        height={yCellHeight}
        fill={colorScale(cell.count)}
        stroke="#fff"
        stroke-width="1"
        onmouseover={() => { hoveredCell = cell; }}
        onmouseout={() => { hoveredCell = null; }}
      />
    {/each}

    {#each coData.genres as genre}
      <text
        x={xScale(genre)! + xCellWidth / 2}
        y={margin.top - 8}
        font-size="10"
        text-anchor="middle"
      >
        {genre}
      </text>
    {/each}

    {#each coData.genres as genre}
      <text
        x={margin.left - 8}
        y={yScale(genre)! + yCellHeight / 2}
        font-size="10"
        text-anchor="end"
      >
        {genre}
      </text>
    {/each}

    {#if hoveredCell && hoveredCell.count > 0}
      <text x={width / 2} y={20} text-anchor="middle" font-size="12">
        {hoveredCell.g1} + {hoveredCell.g2}: {hoveredCell.count}
      </text>
    {/if}
  </svg>
{/if}
