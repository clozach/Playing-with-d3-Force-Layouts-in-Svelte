<script>
  import { onMount } from "svelte";
  import Balloon from "./Balloon.svelte";
  import { initialBalloons } from "./DataSource.svelte";
  import { gridlines } from "./FeatureToggles.svelte";
  import Gridlines from "./Gridlines.svelte";

  let width;
  let height;
  let dataSource = [];
  let simulation; // Can't be initialized before DOM

  // Remember, SVG origin is at the top left, so top is 0.1
  const levelMap = {
    top: 0.1,
    up: 0.3,
    middle: 0.5,
    down: 0.7,
    bottom: 0.9
  };

  const setInitialPositions = balloons => {
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

  const horizontalCenter = d => {
    return width / 2;
  };

  const verticalLevelCenter = d => {
    return 0.8 * height * levelMap[d.height];
  };

  /**
   * Configures the simulation with the forces that position the balloons in their final resting place.
   */
  const setSimulationForces = sim => {
    sim
      .force("charge_force", d3.forceManyBody().strength(-5))
      .force("collisions", d3.forceCollide().radius(30))
      .force("x", d3.forceX().x(horizontalCenter))
      .force("y", d3.forceY().y(verticalLevelCenter));
  };

  const cloneTemplateBalloonInDOM = d => {
    const templateBalloon = document.getElementsByClassName(
      "template-balloon"
    )[0];
    const balloonGroup = document.getElementById("balloon-group");

    var clone = templateBalloon.cloneNode(true);
    clone.getElementsByTagName("text")[0].innerHTML = d.height;
    clone.classList.toggle("template-balloon");
    balloonGroup.append(clone);
  };

  // https://stackoverflow.com/questions/18517376/d3-append-duplicates-of-a-selection @eagor
  function cloneSelection(svg, appendTo, toCopy) {
    toCopy.each(function() {
      // `each` assigns each node to `this`
      var clone = svg.node().appendChild(this.cloneNode(true));
      d3.select(clone)
        .attr("class", "clone")
        .attr("id", "clone-" + i);
    });
    return appendTo.selectAll(".clone");
  }

  const addOne = () => {
    const newData = { id: "⭐️", height: "up" };
    dataSource.push(newData);
    runSim(dataSource);
  };

  /////////////////////////

  const runSim = data => {
    const svg = d3.select("svg");
    simulation.nodes(data);
    data.forEach(cloneTemplateBalloonInDOM);
    const selectedBalloons = bindAndSelectBalloons(svg, data);

    setInitialPositions(selectedBalloons);

    // This function works with `on("tick",…)` to ensure we only
    // call `drop` on the very first tick.
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

    simulation.alpha(1).restart();
  };

  const setUpD3 = () => {
    dataSource = initialBalloons;
    simulation = createSimulation(dataSource);
    setSimulationForces(simulation);

    runSim(dataSource);
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

<button on:click={addOne}>➕🎈</button>

<svg id="billow-field">
  <g id="balloon-group" />
  {#if !!gridlines}
    <g id="gridlines">
      <Gridlines {width} {height} />
    </g>
  {/if}
  <defs>
    <g id="empty-balloon">
      <Balloon id="template" />
    </g>
  </defs>
</svg>
