<script>
  import { onMount } from "svelte";
  import Field from "./Field.svelte";
  import {
    levelMap, // Un-expose this once we're working with real data.
    runSim,
    setSimulationForces
  } from "./FieldFunctions.svelte";
  import { forceSimulation } from "d3";
  import { keypress } from "keypress.js";
  import { initialBalloons } from "./DataSource.svelte";
  import { pickOne } from "./ArrayHelpers.svelte";
  import KeyHint from "./KeyHint.svelte";

  let width;
  let height;

  // The sim needs the `height` to place the levels
  $: {
    simulation = setSimulationForces(forceSimulation(), width, height);
    startSimulation();
  }
  var simulation; // Set after mount
  let datasource = initialBalloons;

  const startup = () => {
    new keypress.Listener().simple_combo("n", addNewBalloon);

    startSimulation();
  };

  const startSimulation = () => {
    runSim(simulation, datasource, {
      startingX: 0,
      startingY: 0,
      transform: "scale(20)"
    });
  };

  const addNewBalloon = () => {
    const newBalloon = {
      id: `${Math.random()}`,
      height: pickOne(Object.keys(levelMap))
    };

    datasource = datasource.concat(newBalloon);
    startSimulation();
  };

  onMount(startup);
</script>

<style>
  #ui-layer {
    width: 100%;
    height: 100%;
  }

  button {
    margin: 1rem;
  }
</style>

<svelte:window bind:innerWidth={width} bind:innerHeight={height} />
<div id="ui-layer">
  <button on:click={addNewBalloon}>
    ➕🎈Add Balloon
    <KeyHint label="[N]" />
  </button>
</div>

<FormEditor />

<Field {width} {height} />
