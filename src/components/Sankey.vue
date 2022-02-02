<template>
  <div ref="sankey"></div>
</template>

<script>
// Code referenced from https://bl.ocks.org/d3noob/31665aced416f27abca4fa46f5f4b568
import * as d3 from "d3";
import csvtojson from "csvtojson";
import csv from "@/data.js";
import { sankey as Sankey, sankeyLinkHorizontal } from "d3-sankey";

export default {
  name: "Sankey",
  props: {
    msg: String,
  },
  async mounted() {
    let jsonData = await csvtojson().fromString(csv);
    let wallets = {
      Polkastarter: "0xee62650fa45ac0deb1b24ec19f983a8f85b727ab",
      PancakeSwap: "0xd6d206f59cc5a3bfa4cc10bc8ba140ac37ad1c89",
      Mint: "0x0000000000000000000000000000000000000000",
      "Primary Wallet": "0x72571d815dd31fbde52be0b9d7ffc8344aede616",
      "Other Wallet": "Other",
      HODL: "HODL",
    };

    let myGraphData = { nodes: [], links: [] };

    for (const [key, value] of Object.entries(wallets)) {
      myGraphData.nodes.push({
        node: value,
        name: key,
      });
    }

    let links = [];
    let HODL = 0;
    jsonData.forEach((entry, index) => {
      let { From, To, Quantity } = entry;
      let createLink = (from, to, value) => ({
        source: from,
        target: to,
        value: Number(value.split(",").join("")),
      });
      let isFromMint = From === wallets.Mint;
      let isFromPriWallet = From === wallets["Primary Wallet"];
      let isFromPolkstarter = From === wallets.Polkastarter;
      let toPriWallet = To === wallets["Primary Wallet"];
      let toPancake = To === wallets.PancakeSwap;
      let toOtherWallets = !toPriWallet && !toPancake;

      let addOrUpdateLink = (toWallet) => {
        let to = toWallet ? toWallet : To;
        let linkIndex = links.findIndex((link) => {
          return link.source === From && link.target === to;
        });
        let isLinkCreated = linkIndex >= 0;
        if (!isLinkCreated) {
          links.push(createLink(From, to, Quantity));
        } else {
          links[linkIndex].value += Number(Quantity.split(",").join(""));
        }
      };

      // Possible Transaction conditions to render in graph.
      let baseConditions =
        (isFromMint && toPriWallet) ||
        (isFromPolkstarter && toPriWallet) ||
        (isFromPolkstarter && toPancake) ||
        (isFromPriWallet && toPancake);

      if (baseConditions) addOrUpdateLink();
      if (
        (isFromPolkstarter && toOtherWallets) ||
        (isFromPriWallet && toOtherWallets)
      ) {
        addOrUpdateLink(wallets["Other Wallet"]);
      }
      if (toPriWallet) {
        HODL += Number(Quantity.split(",").join(""));
      }
      if (isFromPriWallet) {
        if (toOtherWallets || toPancake) {
          HODL -= Number(Quantity.split(",").join(""));
        }
      }
      if (index === jsonData.length - 1) {
        links.push({
          source: wallets["Primary Wallet"],
          target: wallets.HODL,
          value: HODL,
        });
      }

      myGraphData.links = links;
    });

    console.log(myGraphData);

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

    // load the data
    // let data = await d3.json(
    //   "https://raw.githubusercontent.com/holtzy/D3-graph-gallery/master/DATA/data_sankey.json"
    // );

    // let graph = sankey(data);
    let graph = sankey(myGraphData);

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

<!-- Add "scoped" attribute to limit CSS to this component only -->
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
