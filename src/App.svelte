<script>
  import { onMount } from "svelte";
  import {
    timeout,
    forceSimulation,
    forceX,
    forceY,
    forceCollide,
    select
  } from "d3";
  import Balloon from "./Balloon.svelte";
  import { initialBalloons } from "./DataSource.svelte";
  import { gridlines } from "./FeatureToggles.svelte";
  import Gridlines from "./Gridlines.svelte";

  let width;
  let height;

  var simulation; // Set after mount
  let datasource = initialBalloons;

  // Remember, SVG origin is at the top left, so top is 0.1
  const levelMap = {
    top: 0.1,
    up: 0.3,
    middle: 0.5,
    down: 0.7,
    bottom: 0.9
  };

  const drop = balloons => {
    const count = balloons._groups[0].length;
    const totalDuration = 600; // In ms, per usual

    balloons.each((d, i) => {
      timeout(() => {
        d.fx = null;
        d.fy = null;
      }, i * (totalDuration / count));
    });
  };

  const horizontalCenter = d => {
    return width / 2;
  };

  const verticalLevelCenter = d => {
    return 0.8 * height * levelMap[d.height];
  };

  /**
   * Configures the simulation with the forces that position the balloons in their final resting place.
   */
  const setSimulationForces = sim => {
    return sim
      .force("collisions", forceCollide().radius(30))
      .force("x", forceX().x(horizontalCenter))
      .force("y", forceY().y(verticalLevelCenter));
  };

  function balloonCreator() {
    const templateBalloon = document.getElementsByClassName(
      "template-balloon"
    )[0];

    // Inspired by https://stackoverflow.com/questions/18517376/d3-append-duplicates-of-a-selection @eagor
    var clone = templateBalloon.cloneNode(true);
    clone.getElementsByTagName("text")[0].innerHTML = this.__data__.height; // UGH
    clone.classList.toggle("template-balloon");
    return clone;
  }

  function mergeNewData(selection, data) {
    return selection
      .data(data, d => d.id)
      .join(
        enter =>
          enter
            .append(balloonCreator)
            .attr("fx", d => (d.fx = -100))
            .attr("fy", d => (d.fy = 0.8 * height)),
        update => update,
        exit => exit
      );
  }

  function runSim(sim, selection, data) {
    // Without this, the simulation will have no more "energy" when
    // balloons get added after the first run.
    sim.alpha(1);

    sim.nodes(data);

    // This function works with `on("tick",…)` to ensure we only
    // call `drop` on the very first tick.
    var dropper = () => {
      drop(selection);
    };

    sim.on("tick", () => {
      //update balloon positions to reflect node updates on each tick of the simulation
      selection.attr("x", d => d.x);
      selection.attr("y", d => d.y);

      if (dropper) {
        dropper();
        dropper = null;
      }
    });

    sim.restart();
  }
  const startSimulation = () => {
    if (!simulation) {
      simulation = setSimulationForces(forceSimulation());
    }

    const g = select("#balloon-group");
    const balloons = g.selectAll("svg");

    const merged = mergeNewData(balloons, datasource);
    runSim(simulation, merged, datasource);
  };

  const addNewBalloon = () => {
    const pickOne = arr => arr[~~(Math.random() * arr.length)];
    const newBalloon = {
      id: `${Math.random()}`,
      height: pickOne(Object.keys(levelMap))
    };

    datasource = datasource.concat(newBalloon);
    startSimulation();
  };

  onMount(startSimulation);
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

<button on:click={addNewBalloon}>➕🎈Add Balloon (⌘N)</button>

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
