<template>
  <div class="container">
    <h1>Grounding Check</h1>

    <h5>
      Using lisp's lexical conventions, and the backtick-quoted symbol (`) is
      interpreted as the variable prefix.
    </h5>

    <div class="text-box">
      <label for="input1">Rule:</label>
      <textarea
        id="input1"
        v-model="input1"
        placeholder="Rule here"
        @input="updateOutput"
      ></textarea>
    </div>

    <div class="text-box">
      <label for="input2">Fact:</label>
      <textarea
        id="input2"
        v-model="input2"
        placeholder="Fact here"
        @input="updateOutput"
      ></textarea>
    </div>

    <div class="text-box output-box">
      <label for="output">Result:</label>
      <textarea
        id="output"
        v-model="output"
        readonly
        placeholder="Result here"
      ></textarea>
      <div class="status">Update at {{ lastUpdated }}</div>
    </div>
  </div>
</template>

<style scoped>
body {
  font-family: Arial, sans-serif;
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  line-height: 1.6;
  background-color: #f9f9f9;
}
.container {
  background-color: white;
  border-radius: 8px;
  padding: 25px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}
h1 {
  color: #2c3e50;
  text-align: center;
  margin-bottom: 25px;
}
h5 {
  color: #2c3e50;
  text-align: center;
  margin-bottom: 25px;
}
.text-box {
  margin-bottom: 20px;
}
label {
  display: block;
  margin-bottom: 8px;
  font-weight: bold;
  color: #34495e;
}
textarea {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-sizing: border-box;
  min-height: 100px;
  font-family: "Courier New", Courier, monospace;
  font-size: 14px;
  resize: vertical;
  transition: border-color 0.3s;
}
textarea:focus {
  border-color: #3498db;
  outline: none;
  box-shadow: 0 0 0 2px rgba(52, 152, 219, 0.2);
}
.output-box {
  background-color: #f8f9fa;
  border-left: 4px solid #3498db;
}
.status {
  text-align: right;
  font-size: 12px;
  color: #7f8c8d;
  margin-top: 5px;
}
</style>

<script setup>
import { ref, watch } from "vue";
import { rule_t } from "./jsds.mjs";

const input1 = ref(
  "(`P -> `Q)\n" +
  "    `P\n" +
  "----------\n" +
  "    `Q",
);
const input2 = ref("((! (! `X)) -> `X)");
const output = ref("");
const lastUpdated = ref("");

const processStrings = (str1, str2) => {
  let rule1, rule2;
  try {
    rule1 = new rule_t(str1);
    rule2 = new rule_t(str2);
  } catch (error) {
    return "";
  }
  const result = rule1.match(rule2);
  const text = result?.toString();
  return text;
};

const updateOutput = () => {
  const now = new Date();
  lastUpdated.value = now.toLocaleTimeString();
  output.value = processStrings(input1.value, input2.value);
};

watch([input1, input2], updateOutput, { immediate: true });
</script>
