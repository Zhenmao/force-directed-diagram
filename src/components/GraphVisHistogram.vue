<template>
  <div class="chart-wrapper">
    <svg
      class="chart-svg"
      :viewBox="`0 0 ${width} ${height}`"
      :width="width"
      :height="height"
    >
      <g class="bins-g">
        <rect
          v-for="d in rects"
          :key="d.key"
          class="bin-rect"
          :x="x(d.x0)"
          :width="x(d.x1) - x(d.x0)"
          :y="y(d.count)"
          :height="y(0) - y(d.count)"
          :fill="d.color"
        ></rect>
      </g>
      <line
        class="zero-line"
        :x2="width"
        :transform="`translate(0,${height - marginBottom})`"
      ></line>
    </svg>
  </div>
</template>

<script>
import { bin, max, scaleLinear, scaleSequential } from "d3";

export default {
  name: "GraphVisHistogram",
  props: {
    nodes: {
      type: Array,
      default: () => [],
    },
    selectedNodes: {
      type: Array,
      default: () => [],
    },
    colorScheme: {
      type: Function,
      required: true,
    },
  },
  data: () => ({
    width: 224,
    height: 100,
    marginTop: 2,
    marginRight: 4,
    marginBottom: 2,
    marginLeft: 4,
    binsCount: 20,
  }),
  computed: {
    bin() {
      return bin()
        .domain([0, 1])
        .thresholds(this.binsCount)
        .value((d) => d.value);
    },
    bins() {
      return this.bin(this.nodes);
    },
    rects() {
      return this.bins.map((d, i) => {
        const filtered = this.selectedNodes.length
          ? d.filter((e) => this.selectedNodes.includes(e))
          : d;
        return {
          key: i,
          color: this.color((d.x0 + d.x1) / 2),
          count: filtered.length,
          x0: d.x0,
          x1: d.x1,
        };
      });
    },
    x() {
      return scaleLinear()
        .domain([0, 1])
        .range([this.marginLeft, this.width - this.marginRight]);
    },
    yMax() {
      return max(this.bin(this.nodes), (d) => d.length);
    },
    y() {
      return scaleLinear()
        .domain([0, this.yMax])
        .range([this.height - this.marginBottom, this.marginTop]);
    },
    color() {
      return scaleSequential().domain([0, 1]).interpolator(this.colorScheme);
    },
  },
};
</script>

<style scoped>
.zero-line {
  stroke: currentColor;
}
</style>
