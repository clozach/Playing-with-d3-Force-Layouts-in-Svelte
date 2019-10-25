<script context="module">
  import { select, timeout, forceX, forceY, forceCollide } from "d3";
  import { pickOne } from "./ArrayHelpers.svelte";

  /**
   * In my ideal world, Svelte would have some clever way of signaling
   * to the compiler that we expect a function to be pure. Barring
   * that, we can at least make it clear which functions depend
   * exclusively on their arguments, versus those that pull this kind
   * of shenanigan:
   *
   *     var componentVar;
   *     function () { componentVar.modify() }
   *
   * How? By storing these closer-to-pure functions outside of the
   * Svelte component that uses them.
   *
   * (What's the technical term for a function that _can_ modify its
   * arguments…or, shudder, `this`, but can't modify anything else in
   * its calling context?)
   */
  // Remember, SVG origin is at the top left, so top is 0.1
  export const levelMap = {
    top: 0.1,
    up: 0.3,
    middle: 0.5,
    down: 0.7,
    bottom: 0.9
  };

  export const grassHeight = 1 / 10; // Of total height
  export const cloudLayerHeight = 2 / 10; // Of total height

  export const drop = balloons => {
    const count = balloons._groups[0].length;
    const totalDuration = 600; // In ms, per usual

    balloons.each((d, i) => {
      timeout(() => {
        d.fx = null;
        d.fy = null;
      }, i * (totalDuration / count));
    });
  };

  let balloonCount = 0;

  export const newBalloonData = () => {
    balloonCount++;
    return {
      id: balloonCount,
      height: pickOne(Object.keys(levelMap))
    };
  };

  export function balloonCreator() {
    const templateBalloon = document.getElementsByClassName(
      "template-balloon"
    )[0];

    // Inspired by https://stackoverflow.com/questions/18517376/d3-append-duplicates-of-a-selection @eagor
    var clone = templateBalloon.cloneNode(true);
    clone.getElementsByTagName("text")[0].innerHTML = this.__data__.height; // UGH
    clone.classList.toggle("template-balloon");
    clone.classList.toggle("new-balloon");
    return clone;
  }

  export const runEntrySimulation = (sim, data, height, completion) => {
    runSim(
      sim,
      data,
      {
        startingX: -100,
        startingY: height
      },
      completion
    );
  };

  function runSim(sim, data, { startingX, startingY }, completion) {
    function selectionFrom(newdata) {
      const g = select("#balloon-group");
      const balloons = g.selectAll(".new-balloon");

      return balloons
        .data(newdata, d => d.id)
        .join(
          enter =>
            enter
              .append(balloonCreator)
              .attr("fx", d => (d.fx = startingX))
              .attr("fy", d => (d.fy = startingY)),
          // .classed("new-balloon", false),
          update => update,
          exit => exit
        );
    }

    const selection = selectionFrom(data);

    // Without this, the simulation will have no more "energy" when
    // balloons get added after the first run.
    sim.alpha(0.9);
    sim.alphaMin(0.3); // Leave some "room" for the field sim to take over

    // d3-selection and d3-force are completely independent, so we
    // need to let the sim know the data's changed, too.
    sim.nodes(data);

    // This function works with `on("tick",…)` to ensure we only
    // call `drop` on the very first tick per simulation run.
    var dropper = () => {
      drop(selection);
    };

    sim.on("tick", () => {
      // Update balloon positions to reflect node updates on each
      // tick of the simulation
      selection.attr("x", d => d.x);
      selection.attr("y", d => d.y);

      if (dropper) {
        dropper();
        dropper = null;
      }
    });

    sim.on("end", () => {
      selection.classed("new-balloon", false);
      setTimeout(() => {
        completion(selection);
      }, 0);
    });

    sim.restart();
  }

  export const runFieldSimulation = (sim, data) => {
    function selectionFrom(newdata) {
      const g = select("#balloon-group");
      const balloons = g.selectAll("svg");

      return balloons.data(newdata, d => d.id);
    }

    const selection = selectionFrom(data);

    sim.alpha(0.5);

    sim.nodes(data);

    sim.on("tick", () => {
      selection.attr("x", d => d.x);
      selection.attr("y", d => d.y);
    });

    sim.restart();
  };

  /**
   * Configures the simulation with the forces that position the balloons in their final resting place.
   */
  export const setSimulationForces = (
    sim,
    width,
    height,
    {
      alphaTarget,
      alphaMin,
      alphaDecay,
      velocityDecay,
      collisionStrength,
      iterations,
      positionStrength
    } = {
      alphaTarget: 0,
      alphaMin: 0.001,
      alphaDecay: 0.0228,
      velocityDecay: 0.4,
      collisionStrength: 0.7,
      positionStrength: 0.1,
      iterations: 1
    }
  ) => {
    const horizontalCenter = d => {
      return width / 2;
    };

    const verticalLevelCenter = d => {
      // Whaddya know. The 0.04 adjustment seems to line the balloons
      // up perfectly with the grid levels' centers.
      return height * levelMap[d.height] - 0.04 * height;
    };

    return sim
      .alphaTarget(alphaTarget)
      .alphaMin(alphaMin)
      .alphaDecay(alphaDecay)
      .velocityDecay(velocityDecay)
      .force(
        "collisions",
        forceCollide()
          .radius(30)
          .strength(collisionStrength)
          .iterations(iterations)
      )
      .force("x", forceX().x(horizontalCenter))
      .force("y", forceY().y(verticalLevelCenter));
  };
</script>
