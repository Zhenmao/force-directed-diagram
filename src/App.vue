<template>
  <v-app>
    <v-main>
      <v-container>
        <v-row>
          <div cols="12">
            Instruction
            <ul>
              <li>Toggle between lasso and zoom actions.</li>
              <li>
                When the lasso action is selected, left click and drag to select
                nodes.
              </li>
              <li>
                When the zoom action is selected, mouse wheel up/down to zoom
                in/out, left click and drag to pan.
              </li>
            </ul>
          </div>
          <v-col cols="12">
            <ForceDirectedDiagram :graph="graph" @select="selected = $event" />
          </v-col>
          <div cols="12">Selected nodes: {{ selected }}</div>
        </v-row>
      </v-container>
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
