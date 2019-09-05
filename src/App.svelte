<script>
  import { onMount } from "svelte";
  import Balloon from "./Balloon.svelte";

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
    var selectedBalloons = svg
      .select("g")
      .selectAll("svg") // Selects all the <svg>'s under <g>
      .data(nodes_data); // Binds the data to those nodes for use during rendering

    var simulation = d3.forceSimulation().nodes(nodes_data);
    simulation
      .force("charge_force", d3.forceManyBody().strength(-5))
      .force("collisions", d3.forceCollide().radius(24))
      .force("center_force", d3.forceCenter(width / 2 - 30, height / 2 - 30));

    simulation.on("tick", () => {
      //update circle positions to reflect node updates on each tick of the simulation
      selectedBalloons.attr("x", d => d.x);
      selectedBalloons.attr("y", d => d.y);
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
</style>

<svelte:window bind:innerWidth={width} bind:innerHeight={height} />
<svg>
  <g class="nodes">
    {#each nodes_data as d}
      <Balloon id={d.id} />
    {/each}
    }
  </g>
</svg>
