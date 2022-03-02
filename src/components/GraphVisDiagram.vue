<template>
  <div v-resize.quiet="onResize" class="chart-wrapper"></div>
</template>

<script>
import {
  forceCollide,
  forceLink,
  forceManyBody,
  forceSimulation,
  forceX,
  forceY,
  scaleSequential,
  select,
  zoom,
  zoomIdentity,
} from "d3";
import lasso from "@/utilities/lasso";

export default {
  name: "GraphVisDiagram",
  props: {
    graph: {
      type: Object,
      default: () => ({
        nodes: [],
        links: [],
      }),
    },
    action: {
      type: String,
      required: true,
    },
    colorScheme: {
      type: Function,
      required: true,
    },
  },
  data: () => ({
    width: 300,
    height: 150,
    nodeRadius: 5,
    linkWidth: 2,
    transform: zoomIdentity,
    nodes: [],
    links: [],
  }),
  computed: {
    color() {
      return scaleSequential().domain([0, 1]).interpolator(this.colorScheme);
    },
    simulation() {
      return forceSimulation()
        .force("charge", forceManyBody().strength(-200))
        .force(
          "link",
          forceLink()
            .id((d) => d.id)
            .distance(50)
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
        this.links = links;

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
    color() {
      this.render();
    },
  },
  mounted() {
    this.svg = select(this.$el).append("svg").attr("class", "chart-svg");
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
      this.width = this.$el.clientWidth;
      this.height = this.$el.clientHeight;

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
              { stopColor: this.color(d.source.value), offset: "0%" },
              { stopColor: this.color(d.target.value), offset: "100%" },
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
        )
        .attr("transform", (d) => `translate(${d.x},${d.y})`)
        .call((g) =>
          g.select(".node-circle").style("color", (d) => this.color(d.value))
        );

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
.chart-wrapper {
  position: relative;
}

.chart-wrapper >>> .chart-svg {
  display: block;
}

.chart-wrapper >>> .node-circle {
  filter: drop-shadow(0px 0px 2px currentColor);
}

.chart-wrapper >>> .possible .node-circle,
.chart-wrapper >>> .selected .node-circle {
  transform: scale(1.5);
}

.chart-wrapper >>> .lasso path {
  stroke: #1867c0;
  stroke-width: 2px;
}

.chart-wrapper >>> .lasso .drawn {
  fill-opacity: 0.05;
}

.chart-wrapper >>> .lasso .loop_close {
  fill: none;
  stroke-dasharray: 4, 4;
}

.chart-wrapper >>> .lasso .origin {
  fill: #1867c0;
}
</style>
