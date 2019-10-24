<script>
  import { onMount } from "svelte";
  import Field from "./Field.svelte";
  import {
    levelMap, // Un-expose this once we're working with real data.
    startSimulation,
    setSimulationForces,
    newBalloonData
  } from "./FieldFunctions.svelte";
  import { forceSimulation } from "d3";
  import { keypress } from "keypress.js";
  import { initialBalloons } from "./DataSource.svelte";
  import KeyHint from "./KeyHint.svelte";

  let width;
  let height;

  // The sim needs the `height` to place the levels
  $: {
    simulation = setSimulationForces(forceSimulation(), width, height);
    startSimulation(simulation, datasource, height);
  }
  var simulation; // Set after mount
  let datasource = initialBalloons;

  const startup = () => {
    new keypress.Listener().simple_combo("n", addNewBalloon);

    startSimulation(simulation, datasource, height);
  };

  const addNewBalloon = () => {
    datasource = datasource.concat(newBalloonData());
    startSimulation(simulation, datasource, height);
  };

  onMount(startup);
</script>

<svelte:window bind:innerWidth={width} bind:innerHeight={height} />

<button on:click={addNewBalloon}>
  ➕🎈Add Balloon
  <KeyHint label="[N]" />
</button>

<Field {width} {height} />
