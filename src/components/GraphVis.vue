<template>
  <v-card class="graph-vis" :style="{ backgroundColor }" :dark="dark" flat>
    <v-card-title>
      {{ title }}
    </v-card-title>
    <v-card-text>
      <div class="graph-vis-body">
        <div class="graph-vis-controls">
          <GraphVisControlAction v-model="action" :options="actions" />
          <GraphVisControlColorScheme
            v-model="colorScheme"
            :options="colorSchemes"
          />
          <GraphVisControlBackground v-model="background" />
        </div>
        <GraphVisDiagram
          :graph="processedGraph"
          :action="action.value"
          :color-scheme="colorScheme.value"
          @select="onSelect"
        />
        <GraphVisHistogram
          :nodes="processedGraph.nodes"
          :selected-nodes="histogramNodes"
          :color-scheme="colorScheme.value"
        />
      </div>
    </v-card-text>
  </v-card>
</template>

<script>
import {
  interpolateTurbo,
  interpolateRdBu,
  interpolateInferno,
  interpolateViridis,
  interpolateGreys,
} from "d3";

import GraphVisControlAction from "./GraphVisControlAction";
import GraphVisControlColorScheme from "./GraphVisControlColorScheme";
import GraphVisControlBackground from "./GraphVisControlBackground";
import GraphVisDiagram from "./GraphVisDiagram";
import GraphVisHistogram from "./GraphVisHistogram";

const actions = [
  { key: "Lasso", value: "lasso" },
  { key: "Zoom", value: "zoom" },
];

const colorSchemes = [
  {
    key: "Turbo",
    value: interpolateTurbo,
  },
  {
    key: "BlueRed",
    value: (t) => interpolateRdBu(1 - t),
  },
  { key: "Thermal", value: interpolateInferno },
  {
    key: "Viridis",
    value: interpolateViridis,
  },
];

export default {
  name: "GraphVis",
  components: {
    GraphVisControlAction,
    GraphVisControlColorScheme,
    GraphVisControlBackground,
    GraphVisDiagram,
    GraphVisHistogram,
  },
  props: {
    graph: {
      type: Object,
      default: () => ({
        nodes: [],
        links: [],
      }),
    },
    title: {
      type: String,
      default: "",
    },
  },
  data: () => ({
    action: actions[0],
    actions,
    colorScheme: colorSchemes[0],
    colorSchemes,
    background: 0,
    selectedNodes: [],
  }),
  computed: {
    processedGraph() {
      return {
        nodes: this.graph.nodes.map((d) => ({
          id: d.id,
          value: +d.value,
        })),
        links: this.graph.links.map((d) => ({
          source: d.from,
          target: d.to,
        })),
      };
    },
    backgroundColor() {
      return interpolateGreys(this.background);
    },
    dark() {
      return this.background >= 0.5;
    },
    histogramNodes() {
      return this.selectedNodes.length
        ? this.selectedNodes
        : this.processedGraph.nodes;
    },
  },
  methods: {
    onSelect(event) {
      this.$emit("select", event);
      this.selectedNodes = event;
    },
  },
};
</script>

<style scoped>
.graph-vis {
  min-height: 100vh;
  display: grid;
  grid-template-rows: auto 1fr;
}

.graph-vis-body {
  height: 100%;
  display: grid;
  grid-template-rows: auto 1fr 120px;
}

.graph-vis-controls {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  margin: -1rem;
}

.graph-vis-controls > * {
  padding: 1rem;
}
</style>
