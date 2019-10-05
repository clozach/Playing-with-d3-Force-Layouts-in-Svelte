<script context="module">
  import {
    transition,
    select,
    timeout,
    forceX,
    forceY,
    forceCollide
  } from "d3";

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

  export function balloonCreator() {
    const templateBalloon = document.getElementsByClassName(
      "template-balloon"
    )[0];

    // Inspired by https://stackoverflow.com/questions/18517376/d3-append-duplicates-of-a-selection @eagor
    var clone = templateBalloon.cloneNode(true);
    clone.getElementsByTagName("text")[0].innerHTML = this.__data__.height; // UGH
    clone.classList.toggle("template-balloon");
    return clone;
  }

  export function runSim(sim, data, { startingX, startingY }) {
    function selectionFrom(newdata) {
      const g = select("#balloon-group");
      const balloons = g.selectAll("svg");

      const entr = enter => {
        const e = enter
          .append(balloonCreator)
          .attr("fx", d => (d.fx = startingX))
          .attr("fy", d => (d.fy = startingY));

        e.selectAll("g")
          .attr("transform", "scale(20)")
          .transition()
          .duration(1250)
          .attr("transform", "scale(1)");

        return e;
      };

      return balloons
        .data(newdata, d => d.id)
        .join(entr, update => update, exit => exit);
    }

    const selection = selectionFrom(data);

    // Without this, the simulation will have no more "energy" when
    // balloons get added after the first run.
    sim.alpha(1);

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

    sim.restart();
  }

  /**
   * Configures the simulation with the forces that position the balloons in their final resting place.
   */
  export const setSimulationForces = (sim, width, height) => {
    const horizontalCenter = d => {
      return width / 2;
    };

    const verticalLevelCenter = d => {
      // Whaddya know. The 0.04 adjustment seems to line the balloons
      // up perfectly with the grid levels' centers.
      return height * levelMap[d.height] - 0.04 * height;
    };

    return sim
      .force("collisions", forceCollide().radius(30))
      .force("x", forceX().x(horizontalCenter))
      .force("y", forceY().y(verticalLevelCenter));
  };
</script>
