<script>
  import { onMount } from "svelte";

  let width;
  let height;

  var nodes_data = [
    { id: "Brimble", height: 0.1 },
    { id: "Flakips", height: 0.3 },
    { id: "Groppie", height: 0.9 },
    { id: "Czostle", height: 0.5 },
    { id: "Qualxir", height: 0.2 },
    { id: "Merilio", height: 0.7 }
  ];

  const setUpD3 = () => {
    var svg = d3.select("svg");
    var node = svg
      .append("g")
      .attr("class", "nodes")
      .selectAll("unused") // Param detail seems irrelevant here, so far.
      .data(nodes_data)
      .enter()
      .append("path")
      .attr(
        "d",
        () =>
          "M146.231 88C141.59475 149.06325 110.55825 179.34275 78.1905 180.40625C78.71225 180.728 78.98625 181.101 78.85975 181.46825C78.7125 181.8955 78.34075 182.20475 77.83725 182.4085C78.45875 182.785 78.98675 183.0765 79.29625 183.13525C81.0825 183.47525 82.3105 184.2075 82.18 185.1845C81.65975 189.07425 71.359 189.71475 69.8115 186.12075C69.3575 185.06575 70.63625 184.1365 72.54625 183.555C72.969 183.4265 72.994 182.79575 72.984 182.119C72.17475 181.7635 71.567 181.26075 71.41575 180.64475C71.376 180.48325 71.40175 180.33825 71.482 180.20975C39.4925 177.29075 7.890225 146.05275 0.7310525 88C-5.866625 34.5 33.30225 0 73.481 0C113.65975 0 150.0085 38.25 146.231 88z"
      ) // Balloon Shape
      .style("fill", "#C80E0E");

    var simulation = d3.forceSimulation().nodes(nodes_data);
    simulation
      .force("charge_force", d3.forceManyBody())
      .force("collisions", d3.forceCollide().radius(d => 73))
      .force("center_force", d3.forceCenter(width / 2, height / 2));
    simulation.on("tick", () => {
      //update circle positions to reflect node updates on each tick of the simulation
      node.attr("transform", d => {
        return `translate(${d.x},${d.y})`;
      });
    });
  };

  onMount(setUpD3);
</script>

<style>
  svg {
    font-family: "Helvetica", "Arial", sans-serif;
    height: 100%;
    width: 100%;
    background: salmon;
    position: absolute;
    top: 0;
    left: 0;
    z-index: -1;
  }

  h1 {
    color: green;
  }
</style>

<svelte:window bind:innerWidth={width} bind:innerHeight={height} />
<svg />

<h1>Hello</h1>
