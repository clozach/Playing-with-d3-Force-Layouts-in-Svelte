<script>
  import { onMount } from "svelte";
  import Balloon from "./Balloon.svelte";
  import { initialBalloons } from "./DataSource.svelte";

  let width;
  let height;

  // Remember, SVG origin is at the top left, so top is 0.1
  const levelMap = {
    top: 0.1,
    up: 0.3,
    middle: 0.5,
    down: 0.7,
    bottom: 0.9
  };

  const prep = balloons => {
    balloons.attr("fx", d => (d.fx = -100));
    balloons.attr("fy", d => (d.fy = 0.8 * height));
  };

  const drop = balloons => {
    const count = balloons._groups[0].length;
    const totalDuration = 600; // In ms, per usual

    balloons.each((d, i) => {
      d3.timeout(() => {
        d.fx = null;
        d.fy = null;
      }, i * (totalDuration / count));
    });

    balloons
      .selectAll("g")
      .transition()
      .duration(1000)
      .ease(d3.easeSinIn)
      .attrTween("transform", function(d) {
        var i = d3.interpolate(0.5, 1);
        return function(t) {
          return `scale(${i(t)})`;
        };
      });
  };

  /**
   * d3 Combination: binds the given data to Svelte-rendered svgs &
   * returns the equivalent selection.
   */
  const bindAndSelectBalloons = (rootSVG, data) => {
    return rootSVG
      .select("g")
      .selectAll("svg") // Selects all the <svg>'s under <g>
      .data(data); // Binds the data to those nodes for use during rendering
  };

  const createSimulation = data => {
    return d3.forceSimulation().nodes(data);
  };

  /**
   * Impure function. 🙀
   */
  const applyBasicForces = sim => {
    sim
      .force("charge_force", d3.forceManyBody().strength(-5))
      .force("collisions", d3.forceCollide().radius(30))
      .force(
        "x",
        d3.forceX().x(function(d) {
          return width / 2;
        })
      );
  };

  /**
   * Impure function. 🙀
   */
  const applyHeightForce = sim => {
    sim.force(
      "y",
      d3.forceY().y(function(d) {
        return 0.8 * height * levelMap[d.height];
      })
    );
  };

  const setUpD3 = () => {
    const svg = d3.select("svg");
    const selectedBalloons = bindAndSelectBalloons(svg, initialBalloons);

    const simulation = createSimulation(initialBalloons);
    prep(selectedBalloons);
    applyBasicForces(simulation);
    applyHeightForce(simulation);

    var dropper = () => {
      d3.timeout(() => {
        drop(selectedBalloons);
      }, 1000);
    };

    simulation.on("tick", () => {
      //update balloon positions to reflect node updates on each tick of the simulation
      selectedBalloons.attr("x", d => d.x);
      selectedBalloons.attr("y", d => d.y);

      if (dropper) {
        dropper();
        dropper = null;
      }
    });
  };

  onMount(setUpD3);
</script>

<style>
  svg {
    font-family: "Helvetica", "Arial", sans-serif;
    height: 100%;
    width: 100%;
    background: linear-gradient(to top, #ddfdff, #6dd5fa, #2980b9);
    position: absolute;
    top: 0;
    left: 0;
    z-index: -1;
  }
</style>

<svelte:window bind:innerWidth={width} bind:innerHeight={height} />
<svg>
  <g class="nodes">
    {#each initialBalloons as d}
      <Balloon id={d.id} />
    {/each}
    }
  </g>
</svg>
