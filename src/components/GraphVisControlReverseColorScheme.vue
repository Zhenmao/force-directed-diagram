<template>
  <div>
    <v-switch
      v-model="model"
      label="Reverse Color Scheme"
      hide-details
    ></v-switch>
  </div>
</template>

<script>
export default {
  name: "GraphVisControlReverseColorScheme",
  props: {
    value: {
      type: Boolean,
      default: false,
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
  },
};
</script>

<style scoped></style>
