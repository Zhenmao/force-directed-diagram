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
  display: flex;
  align-items: center;
}

.background-color-picker-wrapper >>> .v-slider.theme--light .primary {
  background-color: rgba(0, 0, 0, 0.87) !important;
  border-color: rgba(0, 0, 0, 0.87) !important;
}

.background-color-picker-wrapper
  >>> .v-slider.theme--light
  .v-slider__thumb:before {
  background-color: rgba(0, 0, 0, 0.5);
}

.background-color-picker-wrapper >>> .v-slider.theme--light .primary.lighten-3 {
  background-color: rgba(0, 0, 0, 0.2) !important;
  border-color: rgba(0, 0, 0, 0.2) !important;
}
</style>
