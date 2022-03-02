<template>
  <div>
    <v-btn :loading="loading" @click="download">Download subgroup.csv</v-btn>
  </div>
</template>

<script>
import { csvFormat } from "d3";

export default {
  name: "GraphVisTableControlDownload",
  props: {
    nodes: {
      type: Array,
      default: () => [],
    },
  },
  data: () => ({
    loading: false,
  }),
  methods: {
    download() {
      this.loading = true;
      const csv = csvFormat(this.nodes);
      const blob = new Blob([csv], { type: "text/csv;charset=utf-8;" });
      const link = document.createElement("a");
      if (link.download !== undefined) {
        const url = URL.createObjectURL(blob);
        link.setAttribute("href", url);
        link.setAttribute("download", "subgroup.csv");
        link.style.visibility = "hidden";
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
      }
      this.loading = false;
    },
  },
};
</script>

<style scoped></style>
