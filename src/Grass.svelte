<script>
  import { integers } from "./ArrayHelpers.svelte";
  import { scaleLinear } from "d3-scale";
  import GrassTuft from "./GrassTuft.svelte";

  export let width;
  export let height;

  let tuftWidth = 100; // Based on dimensions in Figma
  let tuftHeight = 98;

  // Adjust these until the tufts form a desirable pattern
  let tuftMargin = tuftWidth ? 0.1 * tuftWidth : 0; // %ge negative margin on either side.
  const tuftHeightMultiplier = 0.1; // 1 == zone twice the height of the grass)

  let scale = scaleLinear().range(-tuftMargin, width + tuftMargin);

  $: tuftZoneHeight = tuftHeight * tuftHeightMultiplier;
  $: tuftCount = Math.ceil(width / tuftWidth) + 10; // Not sure why "+10, but it seems to work!"

  const flipPercentageOfBalloons = perc => {
    const flipHorizontal = gw => {
      return `scale(-1,1) translate(-${gw},0)`;
    };
    return Math.random() > 0.5 ? flipHorizontal(tuftWidth) : "";
  };

  $: y = height - 0.9 * tuftHeight - tuftZoneHeight * Math.random();

  function x(i) {
    return (
      -tuftMargin + (i - 1) * (tuftWidth - 2 * tuftMargin) + 30 * Math.random()
    );
  }
</script>

{#each integers(1, tuftCount) as i}
  <GrassTuft x={x(i)} {y} transform={flipPercentageOfBalloons(0.2)} />
{/each}
