<script>
  import { onMount } from "svelte";
  import { forceSimulation } from "d3";
  import Field from "./Field.svelte";
  import AccessibleButton from "./AccessibleButton.svelte";
  import {
    levelMap, // Un-expose this once we're working with real data.
    runEntrySimulation,
    setSimulationForces,
    newBalloonData
  } from "./FieldFunctions.svelte";
  import { initialBalloons } from "./DataSource.svelte";

  /* -------------------------------------------------------------- */
  /*                              MODEL                             */
  /* -------------------------------------------------------------- */

  let model = initialBalloons;
  let entrySimulation; // Resets whenever the window width & height refresh
  let width;
  let height;

  /* -------------------------------------------------------------- */
  /*                            BINDINGS                            */
  /* -------------------------------------------------------------- */

  $: {
    // The sim needs the `height` to place the levels
    entrySimulation = setSimulationForces(forceSimulation(), width, height);
  }

  const addNewBalloon = () => {
    model = model.concat(newBalloonData());
    runEntrySimulation(entrySimulation, model, height);
  };

  const startup = () => {};

  /* -------------------------------------------------------------- */
  /*                             RUNTIME                            */
  /* -------------------------------------------------------------- */

  onMount(startup);
</script>

<svelte:window bind:innerWidth={width} bind:innerHeight={height} />

<AccessibleButton
  label="➕🎈Add Balloon"
  keyboardTrigger="n"
  action={addNewBalloon} />

<Field {width} {height} />
