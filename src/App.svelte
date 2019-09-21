<script>
  import { onMount } from "svelte";
  import Field from "./Field.svelte";
  import {
    levelMap,
    drop,
    balloonCreator,
    runSim,
    setSimulationForces
  } from "./FieldFunctions.svelte";
  import { timeout, forceSimulation, select } from "d3";
  import { initialBalloons } from "./DataSource.svelte";

  let width;
  let height;

  var simulation; // Set after mount
  let datasource = initialBalloons;

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

  const startSimulation = () => {
    if (!simulation) {
      simulation = setSimulationForces(forceSimulation(), width, height);
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

<svelte:window bind:innerWidth={width} bind:innerHeight={height} />

<button on:click={addNewBalloon}>➕🎈Add Balloon (⌘N)</button>

<Field bind:width bind:height />
