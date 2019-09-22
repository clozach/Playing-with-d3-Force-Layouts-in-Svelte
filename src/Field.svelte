<script>
  import { scaleLinear } from "d3-scale";
  import { integers } from "./ArrayHelpers.svelte";
  import Balloon from "./Balloon.svelte";
  import Cloud from "./Cloud.svelte";
  import { gridlines } from "./FeatureToggles.svelte";
  import Gridlines from "./Gridlines.svelte";

  export let width = 0;
  export let height = 0;

  let cloudWidth = 204; // Based on dimensions in Figma
  let cloudHeight = 114;

  // Adjust these until the clouds form a desirable pattern
  let cloudMargin = cloudWidth ? 0.1 * cloudWidth : 0; // %ge negative margin on either side.
  const cloudHeightMultiplier = 0.3; // 1 == zone twice the height of the clouds)

  $: cloudZoneHeight = cloudHeight * cloudHeightMultiplier;
  $: cloudCount = Math.ceil(width / cloudWidth) + 2; // Not sure why "+2, but it seems to work!"

  const flipPercentageOfBalloons = perc => {
    const flipHorizontal = cw => {
      return `scale(-1,1) translate(-${cw},0)`;
    };
    return Math.random() > 0.2 ? flipHorizontal(cloudWidth) : "";
  };

  let scale = scaleLinear().range(-cloudMargin, width + cloudMargin);
</script>

<style>
  svg {
    font-family: "Helvetica", "Arial", sans-serif;
    height: 100%;
    width: 100%;
    background: linear-gradient(to top, #ddfdff, #6dd5fa, #2980b9);
    position: absolute;
    top: 0;
    left: 0;
    z-index: -1;
  }
</style>

<svg>
  {#each integers(1, cloudCount) as i}
    <Cloud
      x={-cloudMargin + (i - 1) * (cloudWidth - 2 * cloudMargin)}
      y={cloudZoneHeight * Math.random()}
      width={cloudWidth}
      height={cloudHeight}
      transform={flipPercentageOfBalloons(0.2)} />
  {/each}
  <g id="balloon-group" />
  <defs>
    <g id="empty-balloon">
      <Balloon />
    </g>
  </defs>
  {#if !!gridlines}
    <Gridlines {width} {height} />
  {/if}
</svg>
