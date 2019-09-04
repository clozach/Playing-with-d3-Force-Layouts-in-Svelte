<script>
  import { onMount } from "svelte";

  let width;
  let height;

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

    var simulation = d3.forceSimulation().nodes(nodes_data);
    simulation
      .force("charge_force", d3.forceManyBody())
      .force("center_force", d3.forceCenter(width / 2, height / 2));
    simulation.on("tick", () => {
      //update circle positions to reflect node updates on each tick of the simulation
      node
        .attr("cx", d => d.x) // This comment is an annoying hack https://github.com/prettier/prettier-vscode/issues/352
        .attr("cy", d => d.y);
    });
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

<svelte:window bind:innerWidth={width} bind:innerHeight={height} />
<svg />

<h1>Hello</h1>
