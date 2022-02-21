<template>
  <div v-resize.quiet="onResize"></div>
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
    width: 960,
    height: 500,
    nodeRadius: 16,
    linkWidth: 2,
    nodes: [],
    links: [],
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
  },
  mounted() {
    this.svg = select(this.$el).append("svg");
    this.gLink = this.svg
      .append("g")
      .attr("class", "links-g")
      .selectAll(".link-g");
    this.gNode = this.svg
      .append("g")
      .attr("class", "nodes-g")
      .selectAll(".node-g");

    this.lasso.targetArea(this.svg);
    this.svg.call(this.lasso);

    this.onResize();
  },
  methods: {
    onResize() {
      this.width = this.$el.clientWidth;

      this.svg.attr("viewBox", [
        -this.width / 2,
        -this.height / 2,
        this.width,
        this.height,
      ]);

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
  },
};
</script>

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
