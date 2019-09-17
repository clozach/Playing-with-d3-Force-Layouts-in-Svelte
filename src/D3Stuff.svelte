<script context="module">
  // Remember, SVG origin is at the top left, so top is 0.1
  export const levelMap = {
    top: 0.1,
    up: 0.3,
    middle: 0.5,
    down: 0.7,
    bottom: 0.9
  };

  export const createSimulation = data => {
    return d3
      .forceSimulation()
      .alphaDecay(0)
      .nodes(data);
  };

  /* I wasn't able to find documentation on how to create a
     custom force, so I modified this one:

         https://github.com/d3/d3-force/blob/master/src/x.js
     
     to create this jitter function.
  */
  const jitter = () => {
    var nodes = [];

    function force() {
      const jitterMultiplier = 5;
      const randomDistance = () => Math.random() * jitterMultiplier;

      nodes.forEach(n => {
        if (Math.random() > 0.5) {
          n.vx += randomDistance();
        } else {
          n.vx -= randomDistance();
        }
        if (Math.random() > 0.5) {
          n.vy += randomDistance();
        } else {
          n.vy -= randomDistance();
        }
      });
    }

    force.initialize = function(_) {
      nodes = _;
    };

    return force;
  };

  /**
   * Configures the simulation with the forces that position the balloons in their final resting place.
   */
  export const setSimulationForces = (width, height, sim) => {
    // Partial application: verticalLevelCenter(h,d) 👉 centerY(d)
    const centerY = verticalLevelCenter.bind(null, height);

    sim
      .force("charge_force", d3.forceManyBody().strength(-5))
      .force("collisions", d3.forceCollide().radius(30))
      .force("x", d3.forceX().x(horizontalCenter(width)))
      .force("y", d3.forceY().y(centerY))
      .force("jitter", jitter());
  };

  /**
   * d3 Combination: binds the given data to Svelte-rendered svgs &
   * returns the equivalent selection.
   */
  export const bindAndSelectBalloons = (rootSVG, data) => {
    return rootSVG
      .select("g")
      .selectAll("svg") // Selects all the <svg>'s under <g>
      .data(data); // Binds the data to those nodes for use during rendering
  };

  export const cloneTemplateBalloonInDOM = d => {
    const templateBalloon = document.getElementsByClassName(
      "template-balloon"
    )[0];
    const balloonGroup = document.getElementById("balloon-group");

    // Inspired by https://stackoverflow.com/questions/18517376/d3-append-duplicates-of-a-selection @eagor
    var clone = templateBalloon.cloneNode(true);
    clone.getElementsByTagName("text")[0].innerHTML = d.height;
    clone.classList.toggle("template-balloon");
    balloonGroup.append(clone);
  };

  export const setInitialPositions = (balloons, height) => {
    balloons.attr("fx", d => (d.fx = -100));
    balloons.attr("fy", d => (d.fy = 0.8 * height));
  };

  export const drop = balloons => {
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

  export const horizontalCenter = width => {
    return width / 2;
  };

  export const verticalLevelCenter = (height, d) => {
    return 0.8 * height * levelMap[d.height];
  };
</script>
