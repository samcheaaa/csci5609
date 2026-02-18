<script lang="ts">
  import type { TMovie } from "../types";
  import * as d3 from "d3";

  // movies data passed in from the parent page
  let { movies, height = 500 }: { movies: TMovie[]; height?: number } = $props();

  let windowWidth = $state(900);
  let width = $derived(windowWidth - 40);

  // margins
  const margin = { top: 30, bottom: 60, left: 50, right: 150 };
  const chartLeft = margin.left;
  const chartRight = $derived(width - margin.right);
  const chartTop = margin.top;
  const chartBottom = height - margin.bottom;

  // data

  // cleaning year data
  const years = $derived(
    [...new Set(movies.map((m) => {
      return m.year instanceof Date ? m.year.getFullYear() : m.year;
    }))]
      .filter((y) => !isNaN(y))
      .sort((a, b) => a - b)
  );

  // count how many movies per genre, then pick top 3
  const topGenresByYear = $derived.by(() => {
    let result: { year: number; genre: string; count: number }[] = [];

    for (const year of years) {
      let counts: { [genre: string]: number } = {};

      movies.forEach((m) => {
        const movieYear = m.year instanceof Date ? m.year.getFullYear() : m.year;
        if (movieYear === year) {
          m.genres.forEach((g: string) => {
            counts[g] = (counts[g] || 0) + 1;
          });
        }
      });

      const top3 = Object.entries(counts)
        .sort((a, b) => b[1] - a[1])
        .slice(0, 3);

      top3.forEach(([genre, count]) => {
        result.push({ year, genre, count });
      });
    }

    return result;
  });

  // all genres that ever appear in top 3
  const legendGenres = $derived(
    [...new Set(topGenresByYear.map((d) => d.genre))].sort()
  );

  // colors
  const allGenres = $derived(
    [...new Set(movies.flatMap((m) => m.genres))].sort()
  );

  const colorScale = $derived(
    d3.scaleOrdinal()
      .domain(allGenres)
      .range(d3.schemeTableau10)
  );

  // scales

  const xScale = $derived(
    d3.scaleBand()
      .domain(years.map(String))
      .range([chartLeft, chartRight])
      .padding(0.2)
  );

  const xSubScale = $derived(
    d3.scaleBand()
      .domain(["0", "1", "2"])
      .range([0, xScale.bandwidth()])
      .padding(0.05)
  );

  const maxCount = $derived(
    d3.max(topGenresByYear, (d) => d.count) || 0
  );

  const yScale = $derived(
    d3.scaleLinear()
      .domain([0, maxCount])
      .range([chartBottom, chartTop])
  );

  // axis

  let xAxisEl: any = $state();
  let yAxisEl: any = $state();

  $effect(() => {
    if (xAxisEl) {
      d3.select(xAxisEl)
        .call(d3.axisBottom(xScale))
        .selectAll("text")
        .attr("transform", "rotate(45)")
        .style("text-anchor", "start")
        .style("font-size", "10px");
    }
    if (yAxisEl) {
      d3.select(yAxisEl).call(d3.axisLeft(yScale));
    }
  });

  // hover

  let hoveredGenre: string | undefined = $state();

  function getBarsForYear(year: number) {
    return topGenresByYear.filter((d) => d.year === year);
  }
</script>

<svelte:window bind:innerWidth={windowWidth} />

<h3>Q1. Grouped Bar Chart: Yearly Top 3 Genres</h3>

{#if movies.length > 0}
  <svg {width} {height}>

    {#each years as year}
      {#each getBarsForYear(year) as entry, i}
        <rect
          role="img"
          x={xScale(String(year))! + xSubScale(String(i))!}
          y={yScale(entry.count)}
          width={xSubScale.bandwidth()}
          height={yScale(0) - yScale(entry.count)}
          fill={colorScale(entry.genre)}
          opacity={!hoveredGenre || hoveredGenre === entry.genre ? 1 : 0.2}
          onmouseenter={() => (hoveredGenre = entry.genre)}
          onmouseleave={() => (hoveredGenre = undefined)}
        />
      {/each}
    {/each}

    <g transform="translate(0, {chartBottom})" bind:this={xAxisEl} />

    <g transform="translate({chartLeft}, 0)" bind:this={yAxisEl} />

    <text
      transform="rotate(-90)"
      x={-(chartTop + chartBottom) / 2}
      y={15}
      text-anchor="middle"
      font-size="13"
    >
      Movie Count
    </text>

    <!-- Legend -->
    <g transform="translate({chartRight + 20}, {chartTop})">
      {#each legendGenres as genre, i}
        <g
          transform="translate(0, {i * 22})"
          opacity={!hoveredGenre || hoveredGenre === genre ? 1 : 0.2}
          onmouseenter={() => (hoveredGenre = genre)}
          onmouseleave={() => (hoveredGenre = undefined)}
          style="cursor: pointer;"
        >
          <rect width="14" height="14" fill={colorScale(genre)} rx="2" />
          <text x="20" y="12" font-size="12">{genre}</text>
        </g>
      {/each}
    </g>
  </svg>
{/if}

<style>
  rect {
    transition: opacity 0.15s ease;
  }
  svg {
    overflow: visible;
    display: block;
    width: 100%;
  }
</style>