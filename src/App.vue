<template>
  <Header> </Header>
  <component :is="currentView" v-if="currentView" />
</template>

<script setup>
import { ref, computed } from "vue";
import Header from "./Header.vue";
import Search from "./Search.vue";
import Match from "./Match.vue";
const routes = {
  "/search": Search,
  "/match": Match,
};

const currentPath = ref(window.location.hash);
window.addEventListener("hashchange", () => {
  currentPath.value = window.location.hash;
});
const currentView = computed(() => {
  return routes[currentPath.value.slice(1) || "/"] || false;
});
</script>
