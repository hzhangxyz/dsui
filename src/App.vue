<template>
  | <a href="#/">Home</a> | <a href="#/match">Match</a> |
  <a href="#/search">Search</a> | <a href="#/sudoku">Sudoku</a> |
  <component :is="currentView" v-if="currentView" />
</template>

<script setup>
import { ref, computed } from "vue";
import Match from "./Match.vue";
import Search from "./Search.vue";
import Sudoku from "./Sudoku.vue";
const routes = {
  "/match": Match,
  "/search": Search,
  "/sudoku": Sudoku,
};

const currentPath = ref(window.location.hash);
window.addEventListener("hashchange", () => {
  currentPath.value = window.location.hash;
});
const currentView = computed(() => {
  return routes[currentPath.value.slice(1) || "/"] || false;
});
</script>
