<script>
  import { onMount } from "svelte";
  import { forceSimulation } from "d3";
  import Field from "./Field.svelte";
  import {
    levelMap, // Un-expose this once we're working with real data.
    startSimulation,
    setSimulationForces,
    newBalloonData
  } from "./FieldFunctions.svelte";
  import { keypress } from "keypress.js";
  import { initialBalloons } from "./DataSource.svelte";
  import KeyHint from "./KeyHint.svelte";

  /* -------------------------------------------------------------- */
  /*                              MODEL                             */
  /* -------------------------------------------------------------- */

  let model = initialBalloons;
  let simulation; // Resets whenever the window width & height refresh
  let width;
  let height;

  /* -------------------------------------------------------------- */
  /*                            BINDINGS                            */
  /* -------------------------------------------------------------- */

  $: {
    // The sim needs the `height` to place the levels
    simulation = setSimulationForces(forceSimulation(), width, height);
    startSimulation(simulation, model, height);
  }

  const addNewBalloon = () => {
    model = model.concat(newBalloonData());
    startSimulation(simulation, model, height);
  };

  const startup = () => {
    new keypress.Listener().simple_combo("n", addNewBalloon);

    startSimulation(simulation, model, height);
  };

  /* -------------------------------------------------------------- */
  /*                             RUNTIME                            */
  /* -------------------------------------------------------------- */

  onMount(startup);
</script>

<svelte:window bind:innerWidth={width} bind:innerHeight={height} />

<button on:click={addNewBalloon}>
  ➕🎈Add Balloon
  <KeyHint label="[N]" />
</button>

<Field {width} {height} />
