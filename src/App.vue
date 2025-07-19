<template>
  <component :is="currentView" v-if="currentView" />
  <ul v-else>
    <li v-for="(route, path) in routes" :key="path">
      <a :href="`#/${path}`">{{ path }}</a>
    </li>
  </ul>
</template>

<style scoped>
/* 背景和全局字体 */
body {
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  margin: 0;
  padding: 0;
  background: linear-gradient(to right, #ece9e6, #ffffff);
  color: #333;
}

/* 链接列表容器 */
ul {
  list-style: none;
  padding: 2rem;
  max-width: 600px;
  margin: 5rem auto;
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.1);
}

/* 每个路由项 */
li {
  margin: 1rem 0;
}

/* 链接样式 */
a {
  display: block;
  padding: 1rem 1.5rem;
  text-decoration: none;
  background-color: #f0f4f8;
  border-radius: 8px;
  color: #333;
  transition: all 0.3s ease;
  font-weight: 500;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

/* 悬停效果 */
a:hover {
  background-color: #007bff;
  color: white;
  box-shadow: 0 4px 10px rgba(0, 123, 255, 0.2);
}
</style>

<script setup>
import { ref, computed } from "vue";
import match from "./Match.vue";
import search from "./Search.vue";
import sudoku from "./Sudoku.vue";

const routes = { match, search, sudoku };

const currentPath = ref(window.location.hash);
window.addEventListener("hashchange", () => {
  currentPath.value = window.location.hash;
});
const currentView = computed(() => {
  return routes[currentPath.value.slice(2)] || false;
});
</script>
