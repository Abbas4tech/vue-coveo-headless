<template>
  <div class="facet-container">
    <div class="facet-header">
      <h3>{{ title }}</h3>
    </div>
    {{ state.facetValues }}
    <div class="facet-values">
      <div
        v-for="value in state.facetValues"
        :key="value.value"
        class="facet-value"
      >
        <label>
          <input
            type="checkbox"
            :checked="isValueSelected(value)"
            @change="toggleSelect(value)"
          />
          {{ value.value }} ({{ value.numberOfResults }})
        </label>
      </div>
    </div>
    <div class="facet-search" v-if="state.canShowMoreValues">
      <input
        v-model="state.inputValue"
        @input="searchFacetValues"
        placeholder="Search"
      />
      <ul>
        <li
          v-for="result in state.facetSearchResults"
          :key="result.rawValue"
          @click="selectFacetSearchResult(result)"
        >
          {{ result.displayValue }}
        </li>
      </ul>
    </div>
    <div class="facet-buttons">
      <button v-if="state.canShowMoreValues" @click="showMore">
        Show More
      </button>
      <button v-if="state.canShowLessValues" @click="showLess">
        Show Less
      </button>
    </div>
  </div>
</template>

<script lang="ts">
import { reactive, onMounted, onBeforeUnmount } from "vue";
import {
  buildFacet,
  type FacetValue,
  type SpecificFacetSearchResult,
} from "@coveo/headless";
import { headlessEngine } from "@/Engine";

export default {
  name: "atomic-facet",
  props: {
    title: {
      type: String,
      required: true,
    },
    field: {
      type: String,
      required: true,
    },
  },
  setup(props) {
    // Reactive state with correct types
    const state = reactive<{
      facetState: ReturnType<typeof buildFacet>["state"] | null;
      facetValues: FacetValue[];
      facetSearchResults: SpecificFacetSearchResult[];
      inputValue: string;
      canShowMoreValues: boolean;
      canShowLessValues: boolean;
    }>({
      facetState: null,
      facetValues: [],
      facetSearchResults: [],
      inputValue: "",
      canShowMoreValues: false,
      canShowLessValues: false,
    });

    const facet = buildFacet(headlessEngine, {
      options: {
        field: props.field,
        numberOfValues: 5,
      },
    });

    const updateState = () => {
      state.facetState = facet.state;
      state.facetValues = facet.state.values;
      state.canShowMoreValues = facet.state.canShowMoreValues;
      state.canShowLessValues = facet.state.canShowLessValues;
      state.facetSearchResults = facet.state.facetSearch?.values || [];
      console.log(state);
    };

    onMounted(() => {
      facet.subscribe(updateState);
    });

    onBeforeUnmount(() => {
      facet.subscribe(() => {}); // Safely unsubscribe
    });

    const toggleSelect = (value: FacetValue) => {
      facet.toggleSelect(value);
    };

    const isValueSelected = (value: FacetValue) => {
      return facet.isValueSelected(value);
    };

    const showMore = () => {
      facet.showMoreValues();
    };

    const showLess = () => {
      facet.showLessValues();
    };

    const searchFacetValues = () => {
      facet.facetSearch.updateText(state.inputValue);
      facet.facetSearch.search();
    };

    const selectFacetSearchResult = (result: SpecificFacetSearchResult) => {
      facet.facetSearch.select(result);
      state.inputValue = "";
    };

    return {
      state,
      toggleSelect,
      isValueSelected,
      showMore,
      showLess,
      searchFacetValues,
      selectFacetSearchResult,
    };
  },
};
</script>
