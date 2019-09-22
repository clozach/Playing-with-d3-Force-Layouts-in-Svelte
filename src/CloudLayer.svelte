<script>
  import { integers } from "./ArrayHelpers.svelte";
  import { scaleLinear } from "d3-scale";
  import Cloud from "./Cloud.svelte";

  export let width;
  export let height;

  let cloudWidth = 204; // Based on dimensions in Figma
  let cloudHeight = 114;

  // Adjust these until the clouds form a desirable pattern
  let cloudMargin = cloudWidth ? 0.1 * cloudWidth : 0; // %ge negative margin on either side.
  const cloudHeightMultiplier = 0.3; // 1 == zone twice the height of the clouds)

  let scale = scaleLinear().range(-cloudMargin, width + cloudMargin);

  $: cloudZoneHeight = cloudHeight * cloudHeightMultiplier;
  $: cloudCount = Math.ceil(width / cloudWidth) + 2; // Not sure why "+2, but it seems to work!"

  const flipPercentageOfBalloons = perc => {
    const flipHorizontal = cw => {
      return `scale(-1,1) translate(-${cw},0)`;
    };
    return Math.random() > 0.2 ? flipHorizontal(cloudWidth) : "";
  };
</script>

{#each integers(1, cloudCount) as i}
  <Cloud
    x={-cloudMargin + (i - 1) * (cloudWidth - 2 * cloudMargin)}
    y={cloudZoneHeight * Math.random()}
    transform={flipPercentageOfBalloons(0.2)} />
{/each}
