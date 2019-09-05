<script>
  import { onMount } from "svelte";
  import Balloon from "./Balloon.svelte";

  let width;
  let height;

  // Remember, SVG origin is at the top left, so top is 0.1
  const levelMap = {
    top: 0.1,
    up: 0.3,
    middle: 0.5,
    down: 0.7,
    bottom: 0.9
  };

  var nodes_data = [
    { id: "Brimble", height: "bottom" },
    { id: "Flakips", height: "down" },
    { id: "Groppie", height: "top" },
    { id: "Czostle", height: "middle" },
    { id: "Qualxir", height: "bottom" },
    { id: "Merilio", height: "up" },
    { id: "Grunhol", height: "bottom" },
    { id: "Yaaasss", height: "up" },
    { id: "Pouppah", height: "bottom" },
    { id: "Jowdwac", height: "bottom" },
    { id: "Heltchu", height: "bottom" },
    { id: "Flowdar", height: "bottom" },
    { id: "Oeoeooo", height: "bottom" },
    { id: "Nopnope", height: "bottom" },
    { id: "Regreps", height: "up" }
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
      .force("collisions", d3.forceCollide().radius(30))
      .force(
        "y",
        d3.forceY().y(function(d) {
          return 0.8 * height * levelMap[d.height];
        })
      )
      .force(
        "x",
        d3.forceX().x(function(d) {
          return width / 2;
        })
      );

    simulation.on("tick", () => {
      //update balloon positions to reflect node updates on each tick of the simulation
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
    background: linear-gradient(to top, #ddfdff, #6dd5fa, #2980b9);
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
