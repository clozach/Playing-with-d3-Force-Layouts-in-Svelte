<script>
  import { onMount } from "svelte";
  import { forceSimulation } from "d3";
  import Field from "./Field.svelte";
  import AccessibleButton from "./AccessibleButton.svelte";
  import {
    levelMap, // Un-expose this once we're working with real data.
    setSimulationForces,
    newBalloonData,
    runEntrySimulation,
    runFieldSimulation
  } from "./FieldFunctions.svelte";
  import { initialBalloons } from "./DataSource.svelte";

  /* -------------------------------------------------------------- */
  /*                              MODEL                             */
  /* -------------------------------------------------------------- */

  let model = initialBalloons;
  let entrySimulation; // Resets whenever the window width & height refresh
  let fieldSimulation; // Resets whenever the window width & height refresh
  let width;
  let height;

  /* -------------------------------------------------------------- */
  /*                            BINDINGS                            */
  /* -------------------------------------------------------------- */

  $: {
    // The sim needs the `height` to place the levels
    entrySimulation = setSimulationForces(forceSimulation(), width, height);
    fieldSimulation = setSimulationForces(forceSimulation(), width, height);
  }

  const addNewBalloon = () => {
    const newBalloons = [newBalloonData()];
    model = model.concat(newBalloons);
    runEntrySimulation(
      entrySimulation,
      newBalloons,
      height,
      endEntrySelection => {
        runFieldSimulation(fieldSimulation, model, endEntrySelection);
      }
    );
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
