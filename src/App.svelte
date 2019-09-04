<script>
  import { onMount } from "svelte";

  var nodes_data = [
    { id: "Brimble", height: 0.1 },
    { id: "Flakips", height: 0.3 },
    { id: "Groppie", height: 0.9 },
    { id: "Czostle", height: 0.5 },
    { id: "Qualxir", height: 0.2 },
    { id: "Merilio", height: 0.7 }
  ];

  const setUpD3 = () => {
    var svg = d3.select("svg");
    var node = svg
      .append("g")
      .attr("class", "nodes")
      .selectAll("circle")
      .data(nodes_data)
      .enter()
      .append("circle")
      .attr("r", 50)
      .attr("fill", "white");
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
