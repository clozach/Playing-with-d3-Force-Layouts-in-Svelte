<script>
  export let width;
  export let height;

  const heightForLevel = (level, numLevels, maximumHeight) => {
    const levelHeight = maximumHeight / 5;
    return (maximumHeight * level) / 5; // level boundary lines
  };

  const halfHeightForLevel = (level, numLevels, maximumHeight) => {
    const levelHeight = maximumHeight / 5;
    return levelHeight * level - levelHeight / 2; // level center lines
  };
</script>

<svg>
  <g class="gridlines">
    {#if height}
      {#each [1, 2, 3, 4, 5] as level}
        <line
          x1="0"
          y1={heightForLevel(level, 5, height)}
          x2={width}
          y2={heightForLevel(level, 5, height)}
          stroke="black" />
        <text x="20" y={heightForLevel(level, 5, height) - 20}>
          Level {level} | Boundary at {heightForLevel(level, 5, height)}
        </text>
        <line
          x1="0"
          y1={halfHeightForLevel(level, 5, height)}
          x2={width}
          y2={halfHeightForLevel(level, 5, height)}
          stroke="black"
          stroke-dasharray="4" />
        <line
          x1={width / 2}
          x2={width / 2}
          y1="0"
          y2={height}
          stroke="black"
          stroke-dasharray="4" />
      {/each}
    {/if}
    }
  </g>
</svg>
