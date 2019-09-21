<script context="module">
  import { select, timeout, forceX, forceY, forceCollide } from "d3";

  /**
   * While it's a shame not to be able to write provably-pure functions,
   * Elm-like, in Svelte, we can at least make it clear which functions
   * depend exclusively on their arguments, versus those that pull this
   * kind of shenanigan:
   *
   *     var componentVar;
   *     function () { componentVar.modify() }
   *
   * How? By storing these closer-to-pure functions outside of the
   * Svelte component that uses them.
   *
   * (What's the technical term for a function that _can_ modify its
   * arguments…or, shudder, `this`, but can't access anything else not
   * explicitly passed through the visible interface?)
   */
  // Remember, SVG origin is at the top left, so top is 0.1
  export const levelMap = {
    top: 0.1,
    up: 0.3,
    middle: 0.5,
    down: 0.7,
    bottom: 0.9
  };

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

  export function runSim(sim, selection, data) {
    // Without this, the simulation will have no more "energy" when
    // balloons get added after the first run.
    sim.alpha(1);

    sim.nodes(data);

    // This function works with `on("tick",…)` to ensure we only
    // call `drop` on the very first tick.
    var dropper = () => {
      drop(selection);
    };

    sim.on("tick", () => {
      //update balloon positions to reflect node updates on each tick of the simulation
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
      return 0.8 * height * levelMap[d.height];
    };

    return sim
      .force("collisions", forceCollide().radius(30))
      .force("x", forceX().x(horizontalCenter))
      .force("y", forceY().y(verticalLevelCenter));
  };

  export function mergeNewData(data, height) {
    const g = select("#balloon-group");
    const balloons = g.selectAll("svg");

    return balloons
      .data(data, d => d.id)
      .join(
        enter =>
          enter
            .append(balloonCreator)
            .attr("fx", d => (d.fx = -100))
            .attr("fy", d => (d.fy = 0.8 * height)),
        update => update,
        exit => exit
      );
  }
</script>
