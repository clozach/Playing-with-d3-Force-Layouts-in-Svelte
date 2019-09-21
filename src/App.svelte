<script>
  import { onMount } from "svelte";
  import Field from "./Field.svelte";
  import {
    levelMap,
    runSim,
    setSimulationForces,
    mergeNewData
  } from "./FieldFunctions.svelte";
  import { forceSimulation } from "d3";
  import { initialBalloons } from "./DataSource.svelte";
  import { pickOne } from "./ArrayHelpers.svelte";

  let width;
  let height;

  var simulation; // Set after mount
  let datasource = initialBalloons;

  const startSimulation = () => {
    if (!simulation) {
      simulation = setSimulationForces(forceSimulation(), width, height);
    }

    const merged = mergeNewData(datasource, height);
    runSim(simulation, merged, datasource);
  };

  const addNewBalloon = () => {
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
