<template>
  <div class="force-directed-diagram-wrapper">
    <div>
      <v-btn-toggle v-model="action" mandatory>
        <v-btn value="lasso"> Lasso </v-btn>
        <v-btn value="zoom"> Zoom </v-btn>
      </v-btn-toggle>
    </div>
    <div ref="chartContainer" v-resize.quiet="onResize"></div>
  </div>
</template>

<script>
import {
  forceCollide,
  forceLink,
  forceManyBody,
  forceSimulation,
  forceX,
  forceY,
  select,
  zoom,
  zoomIdentity,
} from "d3";
import lasso from "@/utilities/lasso";

export default {
  name: "ForceDirectedDiagram",
  props: {
    graph: {
      type: Object,
      default: () => ({
        nodes: [],
        links: [],
      }),
    },
  },
  data: () => ({
    width: 300,
    height: 150,
    nodeRadius: 16,
    linkWidth: 2,
    nodes: [],
    links: [],
    action: "lasso",
    transform: zoomIdentity,
  }),
  computed: {
    simulation() {
      return forceSimulation()
        .force("charge", forceManyBody().strength(-200))
        .force(
          "link",
          forceLink()
            .id((d) => d.id)
            .distance(80)
        )
        .force("x", forceX())
        .force("y", forceY())
        .force(
          "collision",
          forceCollide().radius(this.nodeRadius).iterations(2)
        )
        .stop();
    },
    lasso() {
      return lasso()
        .closePathSelect(true)
        .closePathDistance(100)
        .on("start", this.lassoStarted)
        .on("draw", this.lassoDrawn)
        .on("end", this.lassoEnded);
    },
    zoom() {
      return zoom()
        .scaleExtent([1 / 16, 16])
        .on("zoom", this.zoomed);
    },
  },
  watch: {
    graph: {
      handler: function ({ nodes, links }) {
        const old = this.gNode
          ? new Map(this.gNode.data().map((d) => [d.id, d]))
          : new Map();
        this.nodes = nodes.map((d) => Object.assign(old.get(d.id) || {}, d));
        this.links = links.map((d) =>
          Object.assign({ source: d.from, target: d.to }, d)
        );

        this.simulation.nodes(this.nodes);
        this.simulation.force("link").links(this.links);
        this.simulation.alpha(1).tick(300);

        this.render();
      },
      immediate: true,
    },
    action() {
      this.actionToggled();
    },
  },
  mounted() {
    this.svg = select(this.$refs.chartContainer).append("svg");
    this.g = this.svg.append("g");
    this.gLink = this.g
      .append("g")
      .attr("class", "links-g")
      .selectAll(".link-g");
    this.gNode = this.g
      .append("g")
      .attr("class", "nodes-g")
      .selectAll(".node-g");

    this.lasso.targetArea(this.svg);
    this.g.call(this.lasso);
    this.svg.call(this.zoom);

    this.onResize();
    this.actionToggled();
  },
  methods: {
    actionToggled() {
      if (this.action === "lasso") {
        this.lasso.enabled(true);
        this.svg.on(".zoom", null);
      } else if (this.action === "zoom") {
        this.lasso.enabled(false);
        this.svg.call(this.zoom);
      }
    },
    onResize() {
      this.width = this.$refs.chartContainer.clientWidth;
      this.height = Math.max(this.$refs.chartContainer.clientHeight, 500);

      this.svg
        .attr("viewBox", [
          -this.width / 2,
          -this.height / 2,
          this.width,
          this.height,
        ])
        .attr("width", this.width)
        .attr("height", this.height);

      this.render();
    },
    render() {
      if (!this.gLink || !this.gNode) return;
      this.gLink = this.gLink
        .data(this.links, (d) => `${d.source.id}-${d.target.id}`)
        .join((enter) =>
          enter
            .append("g")
            .attr("class", "link-g")
            .call((g) =>
              g
                .append("linearGradient")
                .attr("id", (d) => `gradient-${d.source.id}-${d.target.id}`)
                .attr("gradientUnits", "userSpaceOnUse")
            )
            .call((g) =>
              g
                .append("line")
                .attr("class", "link-line")
                .attr(
                  "stroke",
                  (d) => `url('#gradient-${d.source.id}-${d.target.id}')`
                )
                .attr("stroke-width", this.linkWidth)
            )
        )

        .call((g) =>
          g
            .select("linearGradient")
            .attr("x1", (d) => d.source.x)
            .attr("y1", (d) => d.source.y)
            .attr("x2", (d) => d.target.x)
            .attr("y2", (d) => d.target.y)
            .selectAll("stop")
            .data((d) => [
              { stopColor: d.source.color, offset: "0%" },
              { stopColor: d.target.color, offset: "100%" },
            ])
            .join((enter) =>
              enter.append("stop").attr("offset", (d) => d.offset)
            )
            .attr("stop-color", (d) => d.stopColor)
        )
        .call((g) =>
          g
            .select(".link-line")
            .attr("x1", (d) => d.source.x)
            .attr("y1", (d) => d.source.y)
            .attr("x2", (d) => d.target.x)
            .attr("y2", (d) => d.target.y)
        );

      this.gNode = this.gNode
        .data(this.nodes, (d) => d.id)
        .join((enter) =>
          enter
            .append("g")
            .attr("class", "node-g")
            .call((g) =>
              g
                .append("circle")
                .attr("class", "node-circle")
                .attr("fill", "currentColor")
                .attr("r", this.nodeRadius)
            )
            .call((g) =>
              g
                .append("text")
                .attr("class", "node-text")
                .attr("text-anchor", "middle")
                .attr("dy", "0.35em")
                .attr("fill", "#fff")
                .attr("font-size", "12px")
            )
        )
        .attr("transform", (d) => `translate(${d.x},${d.y})`)
        .call((g) => g.select(".node-circle").style("color", (d) => d.color))
        .call((g) => g.select(".node-text").text((d) => d.id));

      this.lasso.items(this.gNode);
    },
    lassoStarted() {
      this.lasso
        .items()
        .classed("not-possible", true)
        .classed("selected", false);
    },
    lassoDrawn() {
      this.lasso
        .possibleItems()
        .classed("not-possible", false)
        .classed("possible", true);

      this.lasso
        .notPossibleItems()
        .classed("not-possible", true)
        .classed("possible", false);
    },
    lassoEnded() {
      this.lasso
        .items()
        .classed("not-possible", false)
        .classed("possible", false);

      this.lasso.selectedItems().classed("selected", true);

      const selectedItemsIds = new Set(
        this.lasso
          .selectedItems()
          .data()
          .map((d) => d.id)
      );
      const selectedNodes = this.graph.nodes.filter((d) =>
        selectedItemsIds.has(d.id)
      );
      this.$emit("select", selectedNodes);
    },
    zoomed(event) {
      this.transform = event.transform;
      this.g.attr("transform", this.transform);
      this.lasso.zoomTransform(this.transform);
    },
  },
};
</script>

<style scoped>
.force-directed-diagram-wrapper {
  height: 100%;
  display: grid;
  grid-template-rows: min-content 1fr;
  gap: 1rem;
}
</style>

<style>
.lasso path {
  stroke: #424242;
  stroke-width: 2px;
}

.lasso .drawn {
  fill-opacity: 0.05;
}

.lasso .loop_close {
  fill: none;
  stroke-dasharray: 4, 4;
}

.lasso .origin {
  fill: #424242;
  fill-opacity: 0.5;
}

.possible .node-circle,
.selected .node-circle {
  stroke: currentColor;
  stroke-opacity: 0.5;
  stroke-width: 10px;
}
</style>
