<template>
  <v-card class="graph-vis" :style="{ backgroundColor }" :dark="dark" flat>
    <v-card-title>
      {{ title }}
    </v-card-title>
    <v-card-text class="graph-vis-body">
      <div class="graph-vis-graph">
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
      <div class="graph-vis-table">
        <GraphVisTable :nodes="tableNodes" :columns="tableColumns" />
        <div class="graph-vis-controls graph-vis-controls--table">
          <GraphVisTableControlLabel
            @label-change="saveLabel"
            v-if="tableNodes.length"
          />
          <GraphVisTableControlDownload :nodes="nodeDetails" class="ml-auto" />
        </div>
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
import GraphVisTable from "./GraphVisTable";
import GraphVisTableControlLabel from "./GraphVisTableControlLabel";
import GraphVisTableControlDownload from "./GraphVisTableControlDownload";

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
    GraphVisTable,
    GraphVisTableControlLabel,
    GraphVisTableControlDownload,
  },
  props: {
    graph: {
      type: Object,
      default: () => ({
        nodes: [],
        links: [],
        nodeDetails: [],
      }),
    },
    title: {
      type: String,
      default: "",
    },
  },
  data: () => ({
    action: actions[1],
    actions,
    colorScheme: colorSchemes[2],
    colorSchemes,
    background: 1,
    selectedNodes: [],
    nodeDetails: [],
  }),
  watch: {
    graph: {
      handler(graph) {
        this.nodeDetails =
          graph.nodeDetails?.map((d) => ({
            ...d,
            label: "",
          })) || [];
      },
      immediate: true,
    },
  },
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
    tableColumns() {
      return this.graph?.nodeDetails?.columns
        ? [...this.graph.nodeDetails.columns, "label"]
        : [];
    },
    selectedNodesIdsSet() {
      return new Set(this.selectedNodes.map((d) => d.id));
    },
    tableNodes() {
      return this.selectedNodes.length
        ? this.nodeDetails.filter((d) =>
            this.selectedNodesIdsSet.has(d.cluster_id)
          )
        : [];
    },
  },
  methods: {
    onSelect(event) {
      this.$emit("select", event);
      this.selectedNodes = event;
    },
    saveLabel(event) {
      this.nodeDetails = this.nodeDetails.map((d) =>
        this.selectedNodesIdsSet.has(d.cluster_id)
          ? {
              ...d,
              label: event,
            }
          : d
      );
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
  display: grid;
  grid-template-rows: 1fr auto;
  gap: 1rem;
}

.graph-vis-graph {
  height: 100%;
  min-height: 600px;
  display: grid;
  grid-template-rows: auto 1fr 120px;
  gap: 1rem;
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
