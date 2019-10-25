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
  let enterFromLeft = false;
  let entrySimulation; // Resets whenever the window width & height refresh
  let fieldSimulation; // Resets whenever the window width & height refresh
  let width;
  let height;

  let fieldAlphaTarget = 0;
  let fieldAlphaMin = 0.001;
  let fieldAlphaDeca = 0.0228;
  let fieldVelocityDecay = 0.4;
  let fieldCollisionStrength = 1;
  let fieldCositionStrength = 0.9;
  let fieldIterations = 9;
  let fieldStartingAlpha = 0.07;

  /* -------------------------------------------------------------- */
  /*                            BINDINGS                            */
  /* -------------------------------------------------------------- */

  $: {
    // The sim needs the `height` to place the levels
    entrySimulation = setSimulationForces(forceSimulation(), width, height);
    fieldSimulation = setSimulationForces(forceSimulation(), width, height, {
      alphaTarget: fieldAlphaTarget,
      alphaMin: fieldAlphaMin,
      alphaDecay: fieldAlphaDeca,
      velocityDecay: fieldVelocityDecay,
      collisionStrength: fieldCollisionStrength,
      positionStrength: fieldCositionStrength,
      iterations: fieldIterations
    });
  }

  const addNewBalloon = () => {
    const newBalloons = [newBalloonData()];
    model = model.concat(newBalloons);
    runEntrySimulation(
      entrySimulation,
      newBalloons,
      {
        startingX: enterFromLeft ? -100 : width + 100,
        startingY: height
      },
      height,
      () => {
        runFieldSimulation(fieldSimulation, model, fieldStartingAlpha);
      }
    );
    enterFromLeft = !enterFromLeft;
  };

  const startup = () => {
    // The `setTimout`, gives us one extra run loop because, from what
    // I can tell, the width and height won't be ready till then,
    // causing the balloons to all start at (0,0) with a shite-ton of
    // warnings. See previous commit to see this in action.
    setTimeout(() => {
      runEntrySimulation(
        fieldSimulation,
        model,
        { startingX: width / 2, startingY: height },
        () => {
          runFieldSimulation(fieldSimulation, model, fieldStartingAlpha);
        }
      );
    }, 0);
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
