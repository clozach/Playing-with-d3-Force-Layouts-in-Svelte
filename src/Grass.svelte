<script>
  import { integers } from "./ArrayHelpers.svelte";
  import GrassTuft from "./GrassTuft.svelte";

  export let width = 0;
  export let height = 0;

  let cloudHeight = 0;
  let cloudWidth = 0;
  let cloudMargin = 0;

  const rawWidth = 100; // Original cloud width in Figma
  const rawHeight = 98; // Original cloud height in Figma
  const cloudWidthOverCloudHeight = rawWidth / rawHeight;
  const layerHeightOverViewHeight = 1 / 10;
  const cloudHeightMultiplier = 1;

  $: layerHeight = height * layerHeightOverViewHeight;
  $: cloudHeight = layerHeight / cloudHeightMultiplier;
  $: cloudWidth = cloudHeight * cloudWidthOverCloudHeight;
  $: cloudScale = cloudHeight / rawHeight;

  // Adjust these until the clouds form a desirable pattern
  $: cloudMargin = cloudWidth ? 0.1 * cloudWidth : 0; // %ge negative margin on either side.

  $: cloudCount = Math.ceil(width / (cloudWidth - 2 * cloudMargin));

  // Returns a horizontal flip transform `perc` percent of the time,
  // where `perc` is a value between 0 and 1.
  //
  // This is the simplest way I could think to keep the clouds from
  // looking too repetetive.
  const flipPercentageOfClouds = perc => {
    const flipHorizontal = cw => {
      return `scale(-1,1) translate(-${cw},0)`;
    };
    return Math.random() > 0.2 ? flipHorizontal(cloudWidth) : "";
  };

  const x = (margin, idx, itemWidth) => {
    return idx * (itemWidth - 2 * margin) - margin;
  };

  const y = (viewHeight, itemHeight) => {
    return viewHeight - itemHeight;
  };

  const transform = (percentToFlip, scale) => {
    return `${flipPercentageOfClouds(percentToFlip)} scale(${scale})`;
  };
</script>

<g id="cloud-group">
  {#each integers(0, cloudCount) as i}
    <GrassTuft
      x={x(cloudMargin, i, cloudWidth)}
      y={y(height, cloudHeight)}
      transform={transform(0.5, cloudScale)} />
  {/each}
</g>
