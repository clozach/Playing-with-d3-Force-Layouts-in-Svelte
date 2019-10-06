<script>
  import Balloon from "./Balloon.svelte";

  export let selectedBalloon = null;
  let scale = "30";
  export let x = 0;
  export let y = -400;
  let width = 0;
  let height = 0;

  $: x = width / 2 - (balloonWidth * scale) / 2;
  $: y = height / 2 - (balloonHeight * scale) / 2;

  $: scale = Math.min(
    (1.2 * width) / balloonWidth,
    (1.2 * height) / balloonHeight
  ); // Slightly bigger than screen

  const balloonWidth = 44; // As currently measured in Chrome. 😬
  const balloonHeight = 56; // As currently measured in Chrome. 😬
</script>

<style>
  svg {
    position: initial;
  }
  svg.foo {
    position: absolute;
    top: 0;
    left: 0;
  }
  input,
  label {
    position: absolute;
    left: 10vw;
  }
  label.x-input {
    top: 50vh;
  }
  label.y-input {
    top: 55vh;
  }
  label.scale-input {
    top: 60vh;
  }
  label.screen {
    top: 65vh;
  }
  line {
    stroke: red;
    stroke-width: 0.3rem;
    stroke-dasharray: 3px;
  }
</style>

{#if selectedBalloon && selectedBalloon.height}
  <svg {width} {height} class="foo" style="background-color:#ff6a0033" {x} {y}>
    <!-- <svg {width} {height} class="foo" style="background-color:#ff6a0033" {x} {y}> -->
    <g>
      <svg overflow="visible" {x} {y} transform="scale(1)">
        <g>
          <Balloon transform="scale({scale})" text={selectedBalloon.height} />
        </g>
      </svg>
      <line x1={x} x2={x} y1="0" y2={height} />
      <line x1="0" x2={width} y1={y} y2={y} />
      <!-- <text {x} {y} transform="scale({scale})">{selectedBalloon.height}</text> -->
    </g>
  </svg>

  <label class="x-input">
    x: {x}
    <input
      style="width:10rem"
      type="range"
      bind:value={x}
      min="-100"
      max={width - balloonWidth * scale}
      step=".01" />
  </label>
  <label class="y-input">
    y: {y}
    <input
      style="width:10rem"
      type="range"
      bind:value={y}
      min="-500"
      max={1.5 * height}
      step=".01" />
  </label>
  <label class="scale-input">
    scale: {scale}
    <input
      style="width:10rem"
      type="range"
      bind:value={scale}
      min="1"
      max={100}
      step=".01" />
  </label>
  <label class="screen">w: {width} h: {height}</label>
{/if}
<svelte:window bind:innerWidth={width} bind:innerHeight={height} />
