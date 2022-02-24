<template>
  <v-app>
    <v-main>
      <GraphVis title="Graph Title" :graph="graph" @select="onSelect" />
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
    Promise.all([csv("/data/d3_nodes.csv"), csv("/data/d3_links.csv")]).then(
      ([nodes, links]) => {
        this.graph = {
          nodes,
          links,
        };
      }
    );
  },
  methods: {
    onSelect(event) {
      console.log(event);
    },
  },
};
</script>
