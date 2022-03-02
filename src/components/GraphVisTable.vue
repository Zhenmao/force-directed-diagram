<template>
  <div class="graph-vis-table-wrapper">
    <v-data-table
      :headers="headers"
      :items="nodes"
      :items-per-page="10"
      no-data-text="No graph nodes selected"
      dense
    ></v-data-table>
  </div>
</template>

<script>
export default {
  name: "GraphVisTable",
  props: {
    nodes: {
      type: Array,
      default: () => [],
    },
    columns: {
      type: Array,
      default: () => [],
    },
  },
  computed: {
    headers() {
      return this.columns.map((column) => ({
        text: column,
        value: column,
        sortable: false,
      }));
    },
  },
};
</script>

<style scoped>
.graph-vis-table-wrapper {
  height: 390px;
}

>>> .v-data-footer {
  justify-content: flex-end;
}

>>> .v-data-footer__select {
  display: none;
}
</style>
