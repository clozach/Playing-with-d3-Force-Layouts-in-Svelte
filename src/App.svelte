<script>
  import { onMount } from "svelte";
  import Balloon from "./Balloon.svelte";
  import { initialBalloons } from "./DataSource.svelte";
  import {
    levelMap,
    bindAndSelectBalloons,
    cloneTemplateBalloonInDOM,
    setInitialPositions,
    drop,
    createSimulation,
    horizontalCenter,
    verticalLevelCenter,
    setSimulationForces
  } from "./D3Stuff.svelte";
  import { gridlines } from "./FeatureToggles.svelte";
  import Gridlines from "./Gridlines.svelte";

  let width;
  let height;
  let dataSource = [];
  let simulation; // Can't be initialized before DOM

  const setUpD3 = () => {
    dataSource = initialBalloons;
    simulation = createSimulation(dataSource);
    setSimulationForces(width, height, simulation);

    runSim(dataSource);
  };

  const addOne = () => {
    const newData = { id: "⭐️", height: "up" };
    dataSource.push(newData);
    runSim(dataSource);
  };

  /////////////////////////

  const runSim = data => {
    simulation.nodes(data);
    data.forEach(cloneTemplateBalloonInDOM);

    const svg = d3.select("svg");
    const selectedBalloons = bindAndSelectBalloons(svg, data);

    setInitialPositions(selectedBalloons, height);

    // This function works with `on("tick",…)` to ensure we only
    // call `drop` on the very first tick.
    var dropper = () => {
      d3.timeout(() => {
        drop(selectedBalloons);
      }, 1000);
    };

    simulation.on("tick", () => {
      //update balloon positions to reflect node updates on each tick of the simulation
      selectedBalloons.attr("x", d => d.x);
      selectedBalloons.attr("y", d => d.y);

      if (dropper) {
        dropper();
        dropper = null;
      }
    });

    simulation.alpha(1).restart();
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

<button on:click={addOne}>➕🎈</button>

<svg id="billow-field">
  <g id="balloon-group" />
  {#if !!gridlines}
    <g id="gridlines">
      <Gridlines {width} {height} />
    </g>
  {/if}
  <defs>
    <g id="empty-balloon">
      <Balloon id="template" />
    </g>
  </defs>
</svg>
