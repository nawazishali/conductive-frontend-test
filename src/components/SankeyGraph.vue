<template>
  <div ref="sankey"></div>
</template>

<script>
// Code referenced from https://bl.ocks.org/d3noob/31665aced416f27abca4fa46f5f4b568
import * as d3 from "d3";
import { sankey as Sankey, sankeyLinkHorizontal } from "d3-sankey";

export default {
  name: "Sankey",
  props: {
    graphData: Object,
  },
  mounted() {
    // console.log(this.graphData)
    // set the dimensions and margins of the graph
    let margin = { top: 10, right: 10, bottom: 10, left: 10 },
      width = 900 - margin.left - margin.right,
      height = 300 - margin.top - margin.bottom;

    // format variables
    let formatNumber = d3.format(",.0f"), // zero decimal places
      format = (d) => formatNumber(d),
      color = d3.scaleOrdinal(d3.schemeCategory10);
    // append the svg object to the body of this component
    let svg = d3
      .select(this.$refs.sankey)
      .append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .append("g")
      .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

    let sankey = Sankey()
      .nodeId((d) => d.node)
      .nodeWidth(20)
      .nodePadding(40)
      .size([width, height]);

    let graph = sankey(this.graphData);

    // add in the links
    let link = svg
      .append("g")
      .selectAll(".link")
      .data(graph.links)
      .enter()
      .append("path")
      .attr("class", "link")
      .attr("d", sankeyLinkHorizontal())
      .attr("stroke-width", (d) => d.width);

    // add the link titles
    link
      .append("title")
      .text(
        (d) => d.source.name + " → " + d.target.name + "\n" + format(d.value)
      );

    // add in the nodes
    let node = svg
      .append("g")
      .selectAll(".node")
      .data(graph.nodes)
      .enter()
      .append("g")
      .attr("class", "node");

    // add the rectangles for the nodes
    node
      .append("rect")
      .attr("x", (d) => d.x0)
      .attr("y", (d) => d.y0)
      .attr("height", (d) => d.y1 - d.y0)
      .attr("width", sankey.nodeWidth())
      .style("fill", (d) => (d.color = color(d.name.replace(/ .*/, ""))))
      .style("stroke", (d) => d3.rgb(d.color).darker(2))
      .append("title")
      .text((d) => d.name + "\n" + format(d.value));

    // add in the title for the nodes
    node
      .append("text")
      .attr("x", (d) => d.x0 - 6)
      .attr("y", (d) => (d.y1 + d.y0) / 2)
      .attr("dy", "0.35em")
      .attr("text-anchor", "end")
      .text((d) => d.name)
      .filter((d) => d.x0 < width / 2)
      .attr("x", (d) => d.x1 + 6)
      .attr("text-anchor", "start");
  },
};
</script>

<style>
.node rect {
  fill-opacity: 0.9;
  shape-rendering: crispEdges;
}

.node text {
  pointer-events: none;
  text-shadow: 0 1px 0 #fff;
}

.link {
  fill: none;
  stroke: #000;
  stroke-opacity: 0.2;
}

.link:hover {
  stroke-opacity: 0.5;
}
</style>
