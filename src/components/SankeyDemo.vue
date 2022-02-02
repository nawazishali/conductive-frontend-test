<template>
  <div>
    <h1>QUIDD Sankey Graph</h1>
    <SankeyGraph v-if="dataProcessed" :graphData="graphData" />
    <div>
      <h3>Transactions in Display: {{NumberOfTransactionsInDisplay}}</h3>
    </div>
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
      NumberOfTransactionsInDisplay: 0,
      totalQuiddsInDisaply: 0,
      wallets: {
        Mint: "0x0000000000000000000000000000000000000000",
        "Primary Wallet": "0x72571d815dd31fbde52be0b9d7ffc8344aede616",
        "Other Wallet": "Other",
        HODL: "HODL",
        PancakeSwap: "0xd6d206f59cc5a3bfa4cc10bc8ba140ac37ad1c89",
        Polkastarter: "0xee62650fa45ac0deb1b24ec19f983a8f85b727ab",
      },
      nodeColors: {
        Mint: "blue",
        "Primary Wallet": "yellow",
        "Other Wallet": "purple",
        HODL: "green",
        PancakeSwap: "red",
        Polkastarter: "royalblue",
      }
    };
  },
  methods: {
    async convertCSVtoJSON() {
      try {
        this.jsonData = await csvtojson().fromString(csv);
      } catch(e) {
        console.log('Error while converting CSV to JSON', e);
        throw e;
      }
    },
    generateNodes() {
      for (const [key, value] of Object.entries(this.wallets)) {
        this.graphData.nodes.push({
          node: value,
          name: key,
          color: this.nodeColors[key]
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
          value: Number(quantity),
        });
      } else {
        links[linkIndex].value += Number(quantity);
      }
      this.NumberOfTransactionsInDisplay++;
    },
    generateLinks() {
      let links = [];
      let HODL = 0;
      this.jsonData.forEach((entry, index) => {
        let { From, To, Quantity } = entry;
        Quantity = Quantity.split(',').join('');

        let isFromMint = From === this.wallets.Mint;
        let isFromPriWallet = From === this.wallets["Primary Wallet"];
        let isFromPolkstarter = From === this.wallets.Polkastarter;
        let toPriWallet = To === this.wallets["Primary Wallet"];
        let toPancake = To === this.wallets.PancakeSwap;
        let toOtherWallets = !toPriWallet && !toPancake;

        // Possible base Transaction conditions to render in graph.
        let baseConditions =
          (isFromMint && toPriWallet) ||
          (isFromPolkstarter && toPriWallet) ||
          (isFromPolkstarter && toPancake) ||
          (isFromPriWallet && toPancake);

        if (baseConditions) this.addOrUpdateLink(From, To, Quantity, links);

        // if transactions are going into other wallets then add link for that.
        if (
          (isFromPolkstarter && toOtherWallets) ||
          (isFromPriWallet && toOtherWallets)
        ) {
          this.addOrUpdateLink(
            From,
            this.wallets["Other Wallet"],
            Quantity,
            links
          );
        }

        // Count all transactions coming into primary wallet
        if (toPriWallet) {
          HODL += Number(Quantity);
        }
        // Subtract outgoing transactions from primary wallet
        if (isFromPriWallet) {
          if (toOtherWallets || toPancake) {
            HODL -= Number(Quantity);
          }
        }
        // finally create the HODL link when the loop ends.
        if (index === this.jsonData.length - 1) {
          this.addOrUpdateLink(
            this.wallets["Primary Wallet"],
            this.wallets.HODL,
            HODL,
            links
          );
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
    },
  },
  async created() {
    await this.initiateProcess();
  },
};
</script>

<style>
</style>
