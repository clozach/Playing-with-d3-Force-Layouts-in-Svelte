<script>
  import { onMount } from "svelte";

  const setUpD3 = () => {
    var svg = d3.select("svg");
    console.log(svg);
  };

  onMount(setUpD3);
</script>

<style>
  svg {
    font-family: "Helvetica", "Arial", sans-serif;
    height: 100%;
    width: 100%;
    background: salmon;
    position: absolute;
    top: 0;
    left: 0;
    z-index: -1;
  }

  h1 {
    color: green;
  }
</style>

<svg />

<h1>Hello</h1>
