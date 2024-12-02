<template>
  <Facet title="Hello" field="filetype" />
  <ResultList></ResultList>
  <Sort></Sort>
</template>

<script lang="ts">
import { loadSearchActions, loadSearchAnalyticsActions } from "@coveo/headless";
import Facet from "./components/Facet.vue";
import { headlessEngine } from "@/Engine";
import ResultList from "./components/ResultList.vue";
import Sort from "./components/Sort.vue";

export default {
  components: {
    Facet,
    ResultList,
    Sort,
  },
  setup() {
    const { logInterfaceLoad } = loadSearchAnalyticsActions(headlessEngine);
    const { executeSearch } = loadSearchActions(headlessEngine);

    headlessEngine.dispatch(executeSearch(logInterfaceLoad()));
  },
};
</script>
