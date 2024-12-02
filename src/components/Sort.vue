<template>
  <div class="sort-container">
    <select @change="handleChange" class="sort-select">
      <option value="relevance">Relevance</option>
      <option value="datedescending">Date Descending</option>
      <option value="dateascending">Date Ascending</option>
    </select>
  </div>
</template>

<script lang="ts">
import { defineComponent, reactive, onMounted, onBeforeUnmount } from "vue";
import {
  buildSort,
  buildRelevanceSortCriterion,
  buildDateSortCriterion,
  SortOrder,
  type Sort as SortType,
  type SortState,
  type SortByRelevancy,
  type SortByDate,
} from "@coveo/headless";
import { headlessEngine } from "@/Engine";

export default defineComponent({
  name: "coveo-sort",
  setup() {
    // Reactive state for the component
    const state = reactive<{
      sortState: SortState | null;
    }>({
      sortState: null,
    });

    // Initialize Sort controller and criteria
    const relevanceSortCriterion: SortByRelevancy =
      buildRelevanceSortCriterion();
    const dateDescendingSortCriterion: SortByDate = buildDateSortCriterion(
      SortOrder.Descending
    );
    const dateAscendingSortCriterion: SortByDate = buildDateSortCriterion(
      SortOrder.Ascending
    );

    const sort: SortType = buildSort(headlessEngine, {
      initialState: {
        criterion: relevanceSortCriterion,
      },
    });

    // Update the state when the Sort controller state changes
    const updateState = () => {
      state.sortState = sort.state;
    };

    onMounted(() => {
      sort.subscribe(updateState);
    });

    onBeforeUnmount(() => {
      sort.subscribe(() => {}); // Cleanup subscription
    });

    // Handle sort change
    const handleChange = (event: Event) => {
      const target = event.target as HTMLSelectElement;
      switch (target.value) {
        case "relevance":
          sort.sortBy(relevanceSortCriterion);
          break;
        case "datedescending":
          sort.sortBy(dateDescendingSortCriterion);
          break;
        case "dateascending":
          sort.sortBy(dateAscendingSortCriterion);
          break;
      }
    };

    return {
      handleChange,
    };
  },
});
</script>

<style scoped>
.sort-container {
  margin: 1rem 0;
}

.sort-select {
  padding: 0.5rem;
  font-size: 1rem;
  border: 1px solid #ccc;
  border-radius: 4px;
}
</style>
