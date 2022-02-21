<template>
  <v-app>
    <v-main>
      <div class="pa-3 full-height-stack">
        <div>
          <ForceDirectedDiagram :graph="graph" @select="selected = $event" />
        </div>
        <div class="lasso-output">Selected nodes: {{ selected }}</div>
      </div>
    </v-main>
  </v-app>
</template>

<script>
import { csv } from "d3";
import ForceDirectedDiagram from "./components/ForceDirectedDiagram";

export default {
  name: "App",
  components: {
    ForceDirectedDiagram,
  },
  data: () => ({
    graph: {
      nodes: [],
      links: [],
    },
    selected: [],
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
};
</script>

<style scoped>
.full-height-stack {
  width: 100%;
  min-height: 100vh;
  display: grid;
  grid-template-rows: 1fr 6rem;
  gap: 2rem;
}

.lasso-output {
  overflow-y: auto;
}
</style>
