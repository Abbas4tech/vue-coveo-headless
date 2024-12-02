<template>
  <ul class="result-list">
    <li v-for="result in results.results" :key="result.uniqueId">
      <div>
        <a :href="result.clickUri" target="_blank">{{ result.title }}</a>
        <hr />
      </div>
    </li>
  </ul>
</template>

<script lang="ts">
import { defineComponent, reactive, onMounted, onBeforeUnmount } from "vue";
import {
  buildResultList,
  type ResultList as ResultListType,
  type Result,
} from "@coveo/headless";
import { headlessEngine } from "@/Engine";

export default defineComponent({
  name: "ResultList",
  setup() {
    const state = reactive<{
      results: Result[];
    }>({
      results: [],
    });

    const headlessResultList: ResultListType = buildResultList(headlessEngine, {
      options: {
        fieldsToInclude: ["date"],
      },
    });

    const updateState = () => {
      state.results = [...headlessResultList.state.results];
      console.log("Updated results:", state.results);
    };

    onMounted(() => {
      headlessResultList.subscribe(updateState);
    });

    onBeforeUnmount(() => {
      headlessResultList.subscribe(() => {});
    });

    console.log(state);
    return {
      results: state,
    };
  },
});
</script>

<style scoped>
.result-list {
  list-style: none;
  padding: 0;
}

.result-list li {
  margin-bottom: 1rem;
}

.result-list a {
  font-weight: bold;
  color: #0078d7;
}

.result-list p {
  margin: 0.5rem 0;
}
</style>
