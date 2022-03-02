<template>
  <div class="background-color-picker-wrapper">
    <v-slider
      v-model="model"
      id="background-color-picker"
      label="Background"
      min="0"
      max="1"
      step="0.01"
      hide-details
    ></v-slider>
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
}
</style>
