<template>
  <div class="action-toggle-wrapper">
    <v-btn-toggle v-model="model" mandatory>
      <v-btn v-for="option in options" :key="option.key" :value="option">
        {{ option.key }}
      </v-btn>
    </v-btn-toggle>
  </div>
</template>

<script>
export default {
  name: "GraphVisControlAction",
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
        return this.value;
      },
      set(value) {
        this.$emit("input", value);
      },
    },
  },
};
</script>

<style scoped>
.action-toggle-wrapper >>> .v-btn-toggle {
  background: none !important;
}
</style>
