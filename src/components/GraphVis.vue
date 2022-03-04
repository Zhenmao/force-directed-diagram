<template>
  <v-sheet class="graph-vis" :style="{ backgroundColor }" :dark="dark">
    <div class="graph-vis-graph pa-4">
      <div class="graph-vis-graph-aside">
        <h1 class="text-h5">{{ title }}</h1>
        <div class="graph-vis-controls graph-vis-controls--graph">
          <GraphVisControlColorScheme
            v-model="colorSchemeType"
            :options="colorSchemeTypes"
          />
          <GraphVisControlReverseColorScheme v-model="reverseColorScheme" />
          <GraphVisControlAction v-model="action" :options="actions" />
          <GraphVisControlBackground v-model="background" />
        </div>
        <GraphVisHistogram
          :nodes="processedGraph.nodes"
          :selected-nodes="histogramNodes"
          :color-scheme="colorScheme"
        />
      </div>
      <div class="graph-vis-graph-main">
        <GraphVisDiagram
          :graph="processedGraph"
          :action="action.value"
          :color-scheme="colorScheme"
          @select="onSelect"
        />
      </div>
    </div>
    <div class="graph-vis-table pa-4">
      <GraphVisTable :nodes="tableNodes" :columns="tableColumns" />
      <div class="graph-vis-controls graph-vis-controls--table">
        <GraphVisTableControlLabel
          @label-change="saveLabel"
          v-if="tableNodes.length"
        />
        <GraphVisTableControlDownload :nodes="nodeDetails" class="ml-auto" />
      </div>
    </div>
  </v-sheet>
</template>

<script>
import {
  interpolateRgb,
  interpolateTurbo,
  interpolateInferno,
  interpolateViridis,
  interpolateGreys,
} from "d3";

import GraphVisControlAction from "./GraphVisControlAction";
import GraphVisControlColorScheme from "./GraphVisControlColorScheme";
import GraphVisControlReverseColorScheme from "./GraphVisControlReverseColorScheme";
import GraphVisControlBackground from "./GraphVisControlBackground";
import GraphVisDiagram from "./GraphVisDiagram";
import GraphVisHistogram from "./GraphVisHistogram";
import GraphVisTable from "./GraphVisTable";
import GraphVisTableControlLabel from "./GraphVisTableControlLabel";
import GraphVisTableControlDownload from "./GraphVisTableControlDownload";

const actions = [
  { key: "Lasso", value: "lasso" },
  { key: "Zoom", value: "zoom" },
  { key: "Shape", value: "shape" },
];

const colorSchemeTypes = [
  {
    key: "Turbo",
    value: interpolateTurbo,
  },
  {
    key: "BlueRed",
    value: interpolateRgb("#0000ff", "#e80018"),
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
    GraphVisControlReverseColorScheme,
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
    colorSchemeType: colorSchemeTypes[2],
    colorSchemeTypes,
    background: 1,
    reverseColorScheme: false,
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
    colorScheme() {
      return this.reverseColorScheme
        ? (t) => this.colorSchemeType.value(1 - t)
        : this.colorSchemeType.value;
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
.graph-vis-graph {
  min-height: 100vh;
}

.graph-vis-graph-main {
  height: 600px;
}

@media (min-width: 1024px) {
  .graph-vis-graph {
    display: grid;
    grid-template-columns: 204px 1fr;
    gap: 1rem;
  }

  .graph-vis-graph-main {
    height: 100%;
  }
}

.graph-vis-graph-aside {
  display: grid;
  gap: 1rem;
  grid-template-rows: auto auto 1fr;
}

.graph-vis-graph-aside > *:last-child {
  margin-top: auto;
}

.graph-vis-controls--table {
  display: flex;
  flex-wrap: wrap;
}

.graph-vis-controls > * + * {
  margin-top: 1rem;
}
</style>
