<script lang="ts">
  import type { TMovie } from "../types";
  import * as d3 from "d3";

  type Props = {
    movies: TMovie[];
    width?: number;
    height?: number;
  };

  let { movies, width = 1400, height = 500 }: Props = $props();
  let selectedGenre: string = $state("");

  const margin = {
    top: 30,
    bottom: 50,
    left: 50,
    right: 120,
  };

  let usableArea = {
    top: margin.top,
    right: width - margin.right,
    bottom: height - margin.bottom,
    left: margin.left,
  };

  function getAnnualTop3(movies: TMovie[]) {
    let yearGenreCount: { [year: number]: { [genre: string]: number } } = {};

    movies.forEach((movie) => {
      const year = movie.year.getFullYear();
      if (!yearGenreCount[year]) yearGenreCount[year] = {};
      movie.genres.forEach((genre) => {
        yearGenreCount[year][genre] = (yearGenreCount[year][genre] || 0) + 1;
      });
    });

    let result: { year: number; genre: string; rank: number }[] = [];

    Object.entries(yearGenreCount).forEach(([yearStr, genres]) => {
      const year = Number(yearStr);
      const sorted = Object.entries(genres)
        .sort((a, b) => b[1] - a[1])
        .slice(0, 3);
      sorted.forEach(([genre], i) => {
        result.push({ year, genre, rank: i + 1 });
      });
    });

    return result;
  }

  const annualTop3 = $derived(getAnnualTop3(movies));
  const years = $derived([...new Set(annualTop3.map((d) => d.year))].sort());
  const allGenres = $derived([...new Set(annualTop3.map((d) => d.genre))]);

  const xScale = $derived(
    d3.scaleLinear()
      .domain(d3.extent(years) as [number, number])
      .range([usableArea.left, usableArea.right])
  );

  const yScale = $derived(
    d3.scalePoint()
      .domain(["1", "2", "3"])
      .range([usableArea.top, usableArea.bottom])
      .padding(0.5)
  );

  const colorScale = $derived(
    d3.scaleOrdinal(d3.schemeTableau10).domain(allGenres)
  );

  function getLines(annualTop3: { year: number; genre: string; rank: number }[]) {
    let grouped: { [genre: string]: { year: number; rank: number }[] } = {};
    
    for (let i = 0; i < annualTop3.length; i++) {
      let item = annualTop3[i];
      let genre = item.genre;
      
      if (!grouped[genre]) {
        grouped[genre] = [];
      }
      grouped[genre].push({ year: item.year, rank: item.rank });
    }

    let result = [];
    
    for (let genre in grouped) {
      let points = grouped[genre].sort((a, b) => a.year - b.year);
      result.push({ genre: genre, points: points });
    }
    
    return result;
  }

  const lines = $derived(getLines(annualTop3));

  function getX(d: { year: number; rank: number }) {
    return xScale(d.year);
  }

  function getY(d: { year: number; rank: number }) {
    return yScale(String(d.rank))!;
  }

  const line = $derived(
    d3.line<{ year: number; rank: number }>()
      .x(getX)
      .y(getY)
  );

  let xAxis: any = $state(),
    yAxis: any = $state();

  function updateAxis() {
    d3.select(xAxis).call(d3.axisBottom(xScale).tickFormat(d3.format("d")));
    d3.select(yAxis).call(d3.axisLeft(yScale));
  }

  $effect(() => {
    updateAxis();
  });

</script>

<h3>Q1: Top 3 Genres Over Time</h3>

{#if movies.length > 0}
  <svg {width} {height}>
    {#each lines as { genre, points }}
      <path
        d={line(points)}
        fill="none"
        stroke={colorScale(genre)}
        stroke-width={selectedGenre === genre ? 4 : 2}
        opacity={selectedGenre === "" || selectedGenre === genre ? 1 : 0.15}
      />
      {#each points as point}
        <circle
          cx={xScale(point.year)}
          cy={yScale(String(point.rank))}
          r={4}
          fill={colorScale(genre)}
          opacity={selectedGenre === "" || selectedGenre === genre ? 1 : 0.15}
          onmouseover={() => { selectedGenre = genre; }}
          onmouseout={() => { selectedGenre = ""; }}
        />
      {/each}
    {/each}

    {#each allGenres as genre, i}
      <g transform="translate({width - margin.right + 10}, {margin.top + i * 20})">
        <rect width="12" height="12" fill={colorScale(genre)} />
        <text x="16" y="10" font-size="11">{genre}</text>
      </g>
    {/each}

    <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
    <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />

    <text x={width / 2} y={height - 5} text-anchor="middle" font-size="13">Year</text>
    <text x={15} y={height / 2} text-anchor="middle" font-size="13"
      transform="rotate(-90, 15, {height / 2})">Rank</text>
  </svg>
{/if}