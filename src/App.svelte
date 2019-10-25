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

  let alphaTarget = 0;
  let alphaMin = 0.001;
  let alphaDecay = 0.0228;
  let velocityDecay = 0.4;
  let collisionStrength = 1;
  let positionStrength = 0.9;
  let iterations = 9;
  let startingAlpha = 0.07;

  /* -------------------------------------------------------------- */
  /*                            BINDINGS                            */
  /* -------------------------------------------------------------- */

  $: {
    // The sim needs the `height` to place the levels
    entrySimulation = setSimulationForces(forceSimulation(), width, height);
    fieldSimulation = setSimulationForces(forceSimulation(), width, height, {
      alphaTarget: alphaTarget,
      alphaMin: alphaMin,
      alphaDecay: alphaDecay,
      velocityDecay: velocityDecay,
      collisionStrength: collisionStrength,
      positionStrength: positionStrength,
      iterations: iterations
    });
  }

  const addNewBalloon = () => {
    const newBalloons = [newBalloonData()];
    model = model.concat(newBalloons);
    runEntrySimulation(entrySimulation, newBalloons, height, () => {
      runFieldSimulation(fieldSimulation, model, startingAlpha);
    });
  };

  const startup = () => {
    runEntrySimulation(entrySimulation, model, height, () => {
      runFieldSimulation(fieldSimulation, model, 1);
    });
  };

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
