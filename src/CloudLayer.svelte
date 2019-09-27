<script>
  import { integers } from "./ArrayHelpers.svelte";
  import { xyt } from "./HorizontalRepeater.svelte";
  import { cloudLayerHeight } from "./FieldFunctions.svelte";
  import Cloud from "./Cloud.svelte";

  export let width = 0;
  export let height = 0;

  const config = {
    defaultItemWidth: 204, // Original cloud width in Figma
    defaultItemHeight: 114, // Original cloud width in Figma
    layerHeightOverViewHeight: cloudLayerHeight,
    verticalSlack: 1.3
  };

  $: positioner = xyt(width, height, config);

  // My first approach was to inline the count like so:
  //
  //    {#each integers(0, positioner.cloudCount()) as i}
  //
  // but it turns out that's not reactive, and so you end up with a
  // count stuck at 0 (because the pre-mounted width is, well, 0).
  $: count = positioner.count;
</script>

<g id="cloud-group">
  {#each integers(0, count()) as i}
    <Cloud
      x={positioner.x(i)}
      y={positioner.y()}
      transform={positioner.transform()} />
  {/each}
</g>
