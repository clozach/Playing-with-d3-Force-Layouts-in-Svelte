<script>
  import Balloon from "./Balloon.svelte";
  import { gridlines } from "./FeatureToggles.svelte";
  import Gridlines from "./Gridlines.svelte";

  export let width = 500;
  export let height = 500;
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
