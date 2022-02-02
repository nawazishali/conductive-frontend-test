<template>
  <div>
    <SankeyGraph v-if="dataProcessed" :graphData="graphData" />
  </div>
</template>

<script>
import csvtojson from "csvtojson";
import csv from "@/data.js";
import SankeyGraph from "@/components/SankeyGraph.vue";

export default {
  name: "SankeyDemo",
  components: {
    SankeyGraph,
  },
  data() {
    return {
      graphData: { nodes: [], links: [] },
      jsonData: {},
      dataProcessed: false,
      wallets: {
        Mint: "0x0000000000000000000000000000000000000000",
        "Primary Wallet": "0x72571d815dd31fbde52be0b9d7ffc8344aede616",
        "Other Wallet": "Other",
        HODL: "HODL",
        PancakeSwap: "0xd6d206f59cc5a3bfa4cc10bc8ba140ac37ad1c89",
        Polkastarter: "0xee62650fa45ac0deb1b24ec19f983a8f85b727ab",
      },
    };
  },
  methods: {
    async convertCSVtoJSON() {
      this.jsonData = await csvtojson().fromString(csv);
    },
    generateNodes() {
      for (const [key, value] of Object.entries(this.wallets)) {
        this.graphData.nodes.push({
          node: value,
          name: key,
        });
      }
    },
    addOrUpdateLink(from, to, quantity, links) {
      let linkIndex = links.findIndex((link) => {
        return link.source === from && link.target === to;
      });
      let isLinkCreated = linkIndex >= 0;
      if (!isLinkCreated) {
        links.push({
          source: from,
          target: to,
          value: Number(quantity.split(",").join("")),
        });
      } else {
        links[linkIndex].value += Number(quantity.split(",").join(""));
      }
    },
    generateLinks() {
      let links = [];
      let HODL = 0;
      this.jsonData.forEach((entry, index) => {
        let { From, To, Quantity } = entry;

        let isFromMint = From === this.wallets.Mint;
        let isFromPriWallet = From === this.wallets["Primary Wallet"];
        let isFromPolkstarter = From === this.wallets.Polkastarter;
        let toPriWallet = To === this.wallets["Primary Wallet"];
        let toPancake = To === this.wallets.PancakeSwap;
        let toOtherWallets = !toPriWallet && !toPancake;

        // Possible Transaction conditions to render in graph.
        let baseConditions =
          (isFromMint && toPriWallet) ||
          (isFromPolkstarter && toPriWallet) ||
          (isFromPolkstarter && toPancake) ||
          (isFromPriWallet && toPancake);

        if (baseConditions) this.addOrUpdateLink(From, To, Quantity, links);
        if (
          (isFromPolkstarter && toOtherWallets) ||
          (isFromPriWallet && toOtherWallets)
        ) {
          this.addOrUpdateLink(From, this.wallets["Other Wallet"], Quantity, links);
        }
        if (toPriWallet) {
          HODL += Number(Quantity.split(",").join(""));
        }
        if (isFromPriWallet) {
          if (toOtherWallets || toPancake) {
            HODL -= Number(Quantity.split(",").join(""));
          }
        }
        if (index === this.jsonData.length - 1) {
          links.push({
            source: this.wallets["Primary Wallet"],
            target: this.wallets.HODL,
            value: HODL,
          });
        }

        this.graphData.links = links;
      });

      // console.log(this.graphData);
    },
    async initiateProcess() {
      await this.convertCSVtoJSON();
      this.generateNodes();
      this.generateLinks();
      this.dataProcessed = true;
    }
  },
  async created() {
    await this.initiateProcess()
  }
};
</script>

<style>
</style>
