<script>
  import { integers } from "./ArrayHelpers.svelte";
  import { xyt } from "./HorizontalRepeater.svelte";
  import GrassTuft from "./GrassTuft.svelte";

  export let width = 0;
  export let height = 0;

  const config = {
    defaultItemWidth: 100, // Original tuft width in Figma
    defaultItemHeight: 98, // Original tuft width in Figma
    layerHeightOverViewHeight: 1 / 10,
    verticalSlack: 1,
    alignTop: false
  };

  $: positioner = xyt(width, height, config);

  // See comment in CloudLayer.svelte re: reactivity here.
  $: count = positioner.count;
</script>

<g id="cloud-group">
  {#each integers(0, count()) as i}
    <GrassTuft
      x={positioner.x(i)}
      y={positioner.y()}
      transform={positioner.transform()} />
  {/each}
</g>
