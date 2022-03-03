<template>
  <div v-resize.quiet="onResize" class="chart-wrapper"></div>
</template>

<script>
import {
  drag,
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
            .distance(60)
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
    drag() {
      return drag()
        .on("start", this.dragStarted)
        .on("drag", this.dragged)
        .on("end", this.dragEnded);
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
    this.link = this.g
      .append("g")
      .attr("class", "links-g")
      .selectAll(".link-g");
    this.node = this.g
      .append("g")
      .attr("class", "nodes-g")
      .selectAll(".node-circle");

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
        this.node
          .classed("is-draggable", false)
          .on(".start", null)
          .on(".drag", null)
          .on(".end", null);
      } else if (this.action === "zoom") {
        this.lasso.enabled(false);
        this.svg.call(this.zoom);
        this.node
          .classed("is-draggable", false)
          .on(".start", null)
          .on(".drag", null)
          .on(".end", null);
      } else if (this.action === "shape") {
        this.lasso.enabled(false);
        this.svg.on(".zoom", null);
        this.node.classed("is-draggable", true).call(this.drag);
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
      if (!this.link || !this.node) return;
      this.link = this.link
        .data(this.links, (d) => `${d.source.id}-${d.target.id}`)
        .join((enter) =>
          enter
            .append("g")
            .attr("class", "link-g")
            .call((g) =>
              g
                .filter((d) => d.source.value !== d.target.value)
                .append("linearGradient")
                .attr("id", (d) => `gradient-${d.source.id}-${d.target.id}`)
                .attr("gradientUnits", "userSpaceOnUse")
            )
            .call((g) =>
              g
                .append("line")
                .attr("class", "link-line")
                .attr("stroke-width", this.linkWidth)
            )
        )
        .call((g) =>
          g
            .filter((d) => d.source.value !== d.target.value)
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
            .attr("stroke", (d) =>
              d.source.value !== d.target.value
                ? `url('#gradient-${d.source.id}-${d.target.id}')`
                : this.color(d.source.value)
            )
            .attr("x1", (d) => d.source.x)
            .attr("y1", (d) => d.source.y)
            .attr("x2", (d) => d.target.x)
            .attr("y2", (d) => d.target.y)
        );

      this.node = this.node
        .data(this.nodes, (d) => d.id)
        .join((enter) =>
          enter
            .append("circle")
            .attr("class", "node-circle")
            .attr("fill", "currentColor")
            .attr("r", this.nodeRadius)
        )
        .attr("transform", (d) => `translate(${d.x},${d.y})`)
        .style("color", (d) => this.color(d.value));

      this.lasso.items(this.node);
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
    dragStarted(event) {
      this.svg.classed("is-dragging", true);
      select(event.sourceEvent.target).classed("is-dragging", true);
    },
    dragged(event) {
      event.subject.x = event.x;
      event.subject.y = event.y;
      this.render();
    },
    dragEnded(event) {
      this.svg.classed("is-dragging", false);
      select(event.sourceEvent.target).classed("is-dragging", false);
    },
  },
};
</script>

<style scoped>
.chart-wrapper {
  position: relative;
  height: 100%;
}

.chart-wrapper >>> .chart-svg {
  display: block;
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
}

/* .chart-wrapper >>> .node-circle {
  filter: drop-shadow(0px 0px 2px currentColor);
} */

.chart-wrapper >>> .node-circle.is-draggable {
  cursor: grab;
}

.chart-wrapper >>> .node-circle.is-dragging,
.chart-wrapper >>> .chart-svg.is-dragging {
  cursor: grabbing;
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
