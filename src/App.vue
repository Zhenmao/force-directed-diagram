<template>
  <v-app>
    <v-main>
      <GraphVis title="Synthesis Graph" :graph="graph" />
    </v-main>
  </v-app>
</template>

<script>
import { csv } from "d3";
import GraphVis from "./components/GraphVis";

export default {
  name: "App",
  components: {
    GraphVis,
  },
  data: () => ({
    graph: {
      nodes: [],
      links: [],
    },
  }),
  created() {
    Promise.all([
      csv("/data/d3_nodes.csv"),
      csv("/data/d3_links.csv"),
      csv("/data/table.csv"),
    ]).then(([nodes, links, nodeDetails]) => {
      Object.freeze(nodes);
      Object.freeze(links);
      Object.freeze(nodeDetails);
      this.graph = {
        nodes,
        links,
        nodeDetails,
      };
    });
  },
};
</script>
