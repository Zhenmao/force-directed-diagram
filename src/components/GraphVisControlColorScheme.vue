<template>
  <div class="color-scheme-select-wrapper">
    <v-select
      :items="options.map((option) => option.key)"
      v-model="model"
      label="Color Scheme"
      filled
      color="primary"
      hide-details
    ></v-select>
  </div>
</template>

<script>
export default {
  name: "GraphVisControlColorScheme",
  props: {
    value: {
      type: Object,
      required: true,
    },
    options: {
      type: Array,
      required: true,
    },
  },
  computed: {
    model: {
      get() {
        return this.value.key;
      },
      set(key) {
        const option = this.options.find((option) => option.key === key);
        this.$emit("input", option);
      },
    },
  },
};
</script>

<style scoped>
.color-scheme-select-wrapper {
  width: auto;
}
</style>
