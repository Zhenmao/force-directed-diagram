<template>
  <div class="background-color-picker-wrapper">
    <v-slider
      v-model="model"
      id="background-color-picker"
      color="primary"
      label="Background"
      min="0"
      max="1"
      step="0.01"
      hide-details
    ></v-slider>
    <div class="swatch" :style="{ background: color }" />
  </div>
</template>

<script>
import { interpolateGreys } from "d3";

export default {
  name: "GraphVisControlBackground",
  props: {
    value: {
      type: Number,
      required: true,
    },
  },
  computed: {
    model: {
      get() {
        return this.value;
      },
      set(value) {
        this.$emit("input", value);
      },
    },
    color() {
      return interpolateGreys(this.model);
    },
  },
};
</script>

<style scoped>
.background-color-picker-wrapper {
  width: 300px;
  display: flex;
  align-items: center;
  margin: 0 -0.25rem;
}

.background-color-picker-wrapper > * {
  margin: 0 0.25rem;
}

.swatch {
  border: 1px solid #1867c0;
  border-radius: 0.25rem;
  width: 24px;
  height: 24px;
}
</style>
