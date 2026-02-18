<script lang="ts">
  import type { TMovie } from "../types";
  import * as d3 from "d3";

  let { movies, width = 700 }: { movies: TMovie[]; width?: number } = $props();

  const step = 28;
  const marginTop = 20;
  const marginRight = 20;
  const marginBottom = 20;
  const marginLeft = 120;

  // genre nodes
  const allGenres = $derived(
    [...new Set(movies.flatMap((m) => m.genres))].sort()
  );

  // height
  const height = $derived(
    (allGenres.length - 1) * step + marginTop + marginBottom
  );

  // links, co-occurance
  const links = $derived.by(() => {
    let counts: { [key: string]: number } = {};

    for (const movie of movies) {
      const genres = movie.genres;
      for (let i = 0; i < genres.length; i++) {
        for (let j = i + 1; j < genres.length; j++) {
          // sort so "Comedy-Drama" and "Drama-Comedy" are the same key
          const key = [genres[i], genres[j]].sort().join("__");
          counts[key] = (counts[key] || 0) + 1;
        }
      }
    }

    return Object.entries(counts).map(([key, value]) => {
      const [source, target] = key.split("__");
      return { source, target, value };
    });
  });

  // colors
  const colorScale = $derived(
    d3.scaleOrdinal()
      .domain(allGenres)
      .range(d3.schemeTableau10)
  );

  // scale
  const yScale = $derived(
    d3.scalePoint()
      .domain(allGenres)
      .range([marginTop, height - marginBottom])
  );

  // arc
  function arcPath(source: string, target: string): string {
    const y1 = yScale(source) ?? 0;
    const y2 = yScale(target) ?? 0;
    const r = Math.abs(y2 - y1) / 2;
    return `M${marginLeft},${y1}A${r},${r} 0,0,${y1 < y2 ? 1 : 0} ${marginLeft},${y2}`;
  }

  // arc stroke width
  const maxLinkValue = $derived(d3.max(links, (d) => d.value) || 1);

  const strokeScale = $derived(
    d3.scaleLinear()
      .domain([1, maxLinkValue])
      .range([1, 6])
  );

  // hover interaction
  let hoveredGenre: string | undefined = $state();

  // checking for links connected to hovered genre
  function isLinkHighlighted(link: { source: string; target: string }): boolean {
    if (!hoveredGenre) return true;
    return link.source === hoveredGenre || link.target === hoveredGenre;
  }

  // checking for genre connected to the hovered genre
  function isGenreHighlighted(genre: string): boolean {
    if (!hoveredGenre) return true;
    if (genre === hoveredGenre) return true;
    return links.some(
      (l) =>
        (l.source === hoveredGenre && l.target === genre) ||
        (l.target === hoveredGenre && l.source === genre)
    );
  }
</script>

<h3>Q2. Arc Diagram: Genre Co-occurrence</h3>

{#if movies.length > 0}
  <svg {width} {height}>

    <g fill="none">
      {#each links as link}
        <path
          d={arcPath(link.source, link.target)}
          stroke={isLinkHighlighted(link) ? colorScale(link.source) : "#ccc"}
          stroke-opacity={isLinkHighlighted(link) ? 0.6 : 0.1}
          stroke-width={strokeScale(link.value)}
        >
        </path>
      {/each}
    </g>

    {#each allGenres as genre}
      <g
        transform="translate({marginLeft}, {yScale(genre)})"
        onmouseenter={() => (hoveredGenre = genre)}
        onmouseleave={() => (hoveredGenre = undefined)}
        style="cursor: pointer;"
      >
        <rect
          x={-marginLeft}
          y={-step / 2}
          width={marginLeft + 40}
          height={step}
          fill="transparent"
        />

        <text
          x="-8"
          dy="0.35em"
          text-anchor="end"
          font-size="12"
          fill={isGenreHighlighted(genre) ? (hoveredGenre === genre ? "#000" : "#333") : "#aaa"}
          font-weight={hoveredGenre === genre ? "bold" : "normal"}
        >
          {genre}
        </text>

        <circle
          r={hoveredGenre === genre ? 5 : 3.5}
          fill={colorScale(genre)}
          opacity={isGenreHighlighted(genre) ? 1 : 0.3}
        />
      </g>
    {/each}
  </svg>
{/if}

<style>
  path {
    transition: stroke-opacity 0.2s ease, stroke 0.2s ease;
  }
  circle {
    transition: opacity 0.2s ease;
  }
  text {
    transition: fill 0.2s ease;
  }
  svg {
    overflow: visible;
  }
</style>