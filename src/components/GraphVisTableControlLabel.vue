<template>
  <form @submit.prevent="labelChange" class="d-flex align-center">
    <v-text-field
      label="Label column name"
      v-model="label"
      class="mr-3"
      filled
      hide-details
    ></v-text-field>
    <v-btn @click="labelChange"> Save Name </v-btn>
  </form>
</template>

<script>
export default {
  name: "GraphVisTableControlLabel",
  data: () => ({
    label: "",
  }),
  methods: {
    labelChange() {
      this.$emit("label-change", this.label);
      this.label = "";
    },
  },
};
</script>

<style scoped></style>
