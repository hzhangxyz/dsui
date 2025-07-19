<template>
  <div class="sudoku-container">
    <h1>Sudoku Solver</h1>
    <div class="sudoku-grid">
      <div v-for="(row, rowIndex) in grid" :key="rowIndex">
        <span v-for="(cell, colIndex) in row" :key="colIndex">
          <input
            type="text"
            v-model="grid[rowIndex][colIndex]"
            @focus="focusInput"
            @blur="blueInput(rowIndex, colIndex)"
            @keydown="keyDownInput($event, rowIndex, colIndex)"
            :ref="(el) => refInput(el, rowIndex, colIndex)"
            :readonly="run"
          />
        </span>
      </div>
    </div>

    <button @click="solveSudoku()">Solve</button>
    <button @click="updateSudoku()">Update</button>
    <button @click="clearSudoku()">Clear</button>

    <h3>Log:</h3>
    <textarea v-model="log" readonly></textarea>
  </div>
</template>

<style scoped>
.sudoku-container {
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  background-color: #f0f2f5;
  min-height: 100vh;
  box-sizing: border-box;
}

h1 {
  color: #2c3e50;
  margin-bottom: 30px;
  font-size: 2.5em;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.1);
}

.sudoku-grid {
  display: grid;
  grid-template-columns: repeat(9, 1fr); /* 确保是 9 列 */
  grid-template-rows: repeat(9, 1fr); /* 确保是 9 行 */
  border: 4px solid #34495e; /* 整体边框 */
  border-radius: 8px;
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
  margin-bottom: 30px;
  background-color: #fff; /* 默认背景，会被单元格覆盖 */
}

.sudoku-grid > div {
  /* Vue 的每一行 div */
  display: contents; /* 让子元素直接参与父级 grid 布局 */
}

.sudoku-grid span {
  /* 每个单元格的容器 */
  display: flex;
  justify-content: center;
  align-items: center;
  width: 50px; /* 单元格大小 */
  height: 50px;
  border: 1px solid #ccc; /* 默认单元格细边框 */
  box-sizing: border-box;
}

/* 粗边框：模拟 3x3 宫格分隔线 */

/* 垂直粗边框 */
.sudoku-grid span:nth-child(3n) {
  /* 每行的第 3, 6, 9 个单元格 */
  border-right: 2px solid #95a5a6;
}
.sudoku-grid span:nth-child(9n) {
  /* 每行的第 9 个单元格，也是总网格右边框的一部分 */
  border-right: 4px solid #34495e;
}
.sudoku-grid span:nth-child(9n + 1) {
  /* 每行的第 1 个单元格，总网格左边框的一部分 */
  border-left: 4px solid #34495e;
}

/* 水平粗边框 */
.sudoku-grid > div:nth-child(3n) span {
  /* 第 3, 6, 9 行的单元格 */
  border-bottom: 2px solid #95a5a6;
}
.sudoku-grid > div:nth-child(9) span {
  /* 第 9 行的单元格，总网格下边框的一部分 */
  border-bottom: 4px solid #34495e;
}
.sudoku-grid > div:nth-child(1) span {
  /* 第 1 行的单元格，总网格上边框的一部分 */
  border-top: 4px solid #34495e;
}

/* ------------------------------------------------------------- */
/* 核心修正：3x3 宫格背景交错 */
/* ------------------------------------------------------------- */

/* 第一组 3x3 宫格 (浅色背景 #f9f9f9) */
/* 包含的宫格位置: (1,1), (1,3), (2,2), (3,1), (3,3) */
/* 对应的单元格索引需要精确计算 */
.sudoku-grid span:nth-child(1n) {
  /* 默认给所有 span 一个基础背景 */
  background-color: #e9e9e9; /* 默认较深的背景 */
}

/* 覆盖为浅色背景的单元格 */
/* 目标： 
   (row 1-3, col 1-3)  - 宫格 (1,1)
   (row 1-3, col 7-9)  - 宫格 (1,3)
   (row 4-6, col 4-6)  - 宫格 (2,2)
   (row 7-9, col 1-3)  - 宫格 (3,1)
   (row 7-9, col 7-9)  - 宫格 (3,3)
*/
.sudoku-grid > div:nth-child(-n + 3) span:nth-child(-n + 3), /* Rows 1-3, Cols 1-3 */
.sudoku-grid > div:nth-child(-n + 3) span:nth-child(n + 7), /* Rows 1-3, Cols 7-9 */
.sudoku-grid > div:nth-child(n + 4):nth-child(-n + 6) span:nth-child(n + 4):nth-child(-n + 6), /* Rows 4-6, Cols 4-6 */
.sudoku-grid > div:nth-child(n + 7) span:nth-child(-n + 3), /* Rows 7-9, Cols 1-3 */
.sudoku-grid > div:nth-child(n + 7) span:nth-child(n + 7) /* Rows 7-9, Cols 7-9 */ {
  background-color: #f9f9f9; /* 浅色背景 */
}

/* 输入框样式 */
.sudoku-grid input {
  width: 100%;
  height: 100%;
  border: none;
  text-align: center;
  font-size: 1.8em;
  font-weight: bold;
  color: #333;
  background-color: transparent; /* 背景由父级 span 控制 */
  outline: none;
  transition: background-color 0.2s ease-in-out;
}

.sudoku-grid input:focus {
  background-color: #e8f0fe; /* 选中时蓝色背景 */
}

/* 确保焦点和只读背景覆盖掉默认的宫格背景 */
.sudoku-grid input:focus {
  background-color: #e8f0fe !important; /* !important 确保优先级 */
}

/* 按钮样式 (与之前相同) */
button {
  background-color: #3498db;
  color: white;
  border: none;
  border-radius: 5px;
  padding: 12px 25px;
  margin: 0 10px 20px;
  font-size: 1.1em;
  cursor: pointer;
  transition:
    background-color 0.3s ease,
    transform 0.2s ease;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

button:hover {
  background-color: #2980b9;
  transform: translateY(-2px);
}

button:active {
  background-color: #2980b9;
  transform: translateY(0);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

button:nth-of-type(1) {
  background-color: #2ecc71;
}
button:nth-of-type(1):hover {
  background-color: #27ae60;
}

button:nth-of-type(2) {
  background-color: #f39c12;
}
button:nth-of-type(2):hover {
  background-color: #e67e22;
}

button:nth-of-type(3) {
  background-color: #e74c3c;
}
button:nth-of-type(3):hover {
  background-color: #c0392b;
}

h3 {
  color: #34495e;
  margin-top: 20px;
  margin-bottom: 10px;
  font-size: 1.5em;
}

textarea {
  width: 80%;
  max-width: 600px;
  height: 150px;
  padding: 15px;
  border: 1px solid #ccc;
  border-radius: 8px;
  font-size: 1em;
  font-family: "Consolas", "Monaco", monospace;
  background-color: #ecf0f1;
  color: #34495e;
  resize: vertical;
  box-shadow: inset 0 1px 3px rgba(0, 0, 0, 0.1);
  line-height: 1.5;
}

textarea:focus {
  border-color: #3498db;
  box-shadow:
    inset 0 1px 3px rgba(0, 0, 0, 0.1),
    0 0 8px rgba(52, 152, 219, 0.5);
}
</style>

<script setup>
import { ref, useTemplateRef } from "vue";
import { rule_t, buffer_size } from "./jsds.mjs";
import rule_data_url from "./Sudoku.dat";

let grid = ref(
  Array.from({ length: 9 }, () => Array.from({ length: 9 }, () => "")),
);
let log = ref("");
let run = ref(false);
let inputRef = ref(
  Array.from({ length: 9 }, () => Array.from({ length: 9 }, () => null)),
);

function focusInput(event) {
  event.target.select();
}

function blueInput(rowIndex, colIndex) {
  const value = grid.value[rowIndex][colIndex];
  if (!["", "1", "2", "3", "4", "5", "6", "7", "8", "9"].includes(value)) {
    grid.value[rowIndex][colIndex] = "";
  }
}

function keyDownInput(event, row, col) {
  const { key } = event;
  let newRow = row;
  let newCol = col;
  switch (key) {
    case "ArrowUp":
      newRow = Math.max(0, row - 1);
      break;
    case "ArrowDown":
      newRow = Math.min(8, row + 1);
      break;
    case "ArrowLeft":
      newCol = Math.max(0, col - 1);
      break;
    case "ArrowRight":
      newCol = Math.min(8, col + 1);
      break;
    default:
      return;
  }
  event.preventDefault();
  const nextInput = inputRef.value[newRow][newCol];
  if (nextInput) {
    nextInput.focus();
  }
}

function refInput(el, row, col) {
  inputRef.value[row][col] = el;
}

let generator = null;

function clearSudoku() {
  grid.value = Array.from({ length: 9 }, () =>
    Array.from({ length: 9 }, () => ""),
  );
  log.value = "";
  run.value = false;
  generator = null;
}

async function solveSudoku() {
  if (!generator) {
    generator = search();
  }

  while (true) {
    if ((await generator.next()).done) {
      addLog("Sudoku solved.");
      break;
    }
  }
}

async function updateSudoku() {
  if (!generator) {
    generator = search();
  }

  if ((await generator.next()).done) {
    addLog("Search completed.");
  } else {
    addLog("Search not completed...");
  }
}

async function* search() {
  addLog("Loading Sudoku grid...");
  run.value = true;
  const response = await fetch(rule_data_url);
  const text = await response.text();
  const sections = text.split(/\n\n/);
  let data = sections
    .filter((section) => section.trim().length > 0)
    .map((section) => section.trim());
  for (let row = 0; row < 9; row++) {
    for (let col = 0; col < 9; col++) {
      const value = grid.value[row][col];
      if (["1", "2", "3", "4", "5", "6", "7", "8", "9"].includes(value)) {
        addLog(`Cell (${row + 1}, ${col + 1}) = ${value}`);
        data.push(`((Cell ${row + 1} ${col + 1}) = (Literal ${value}))`);
      }
    }
  }
  addLog("Search start...");
  yield* engine(data, 1000, (candidate) => {
    if (candidate.length() === 0) {
      const text = candidate.toString();
      const match = text.match(/\(\(Cell (\d) (\d)\) = \(Literal (\d)\)\)/);
      if (match) {
        const row = parseInt(match[1], 10);
        const col = parseInt(match[2], 10);
        const value = match[3];
        addLog(`Cell (${row}, ${col}) = ${value}`);
        grid.value[row - 1][col - 1] = value;
      }
    }
  });
}

function addLog(message) {
  log.value += message + "\n";
  const logArea = document.querySelector("textarea");
  logArea.scrollTop = logArea.scrollHeight;
}

function* engine(input_strings, buffer_limit, callback) {
  buffer_size(buffer_limit);

  let rules = {};
  let facts = {};

  let cycle = -1;

  for (let input_string of input_strings) {
    let rule = new rule_t(input_string);
    if (rule.length() !== 0) {
      rules[rule.key()] = [rule, cycle];
    } else {
      facts[rule.key()] = [rule, cycle];
    }
  }

  while (true) {
    let temp_rules = {};
    let temp_facts = {};
    let something_new = false;

    for (let r_hash in rules) {
      for (const f_hash in facts) {
        const [rule, r_cycle] = rules[r_hash];
        const [fact, f_cycle] = facts[f_hash];
        if (r_cycle !== cycle && f_cycle !== cycle) {
          continue;
        }
        const candidate = rule.match(fact);
        if (candidate === null) {
          continue;
        }
        const candidate_hash = candidate.key();
        if (candidate.length() !== 0) {
          // rule
          if (candidate_hash in rules || candidate_hash in temp_rules) {
            continue;
          }
          temp_rules[candidate_hash] = candidate;
        } else {
          // fact
          if (candidate_hash in facts || candidate_hash in temp_facts) {
            continue;
          }
          temp_facts[candidate_hash] = candidate;
        }
        callback(candidate);
        something_new = true;
      }
    }

    cycle++;
    for (let r_hash in temp_rules) {
      const rule = temp_rules[r_hash];
      rules[rule.key()] = [rule, cycle];
    }
    for (let f_hash in temp_facts) {
      const fact = temp_facts[f_hash];
      facts[fact.key()] = [fact, cycle];
    }
    if (!something_new) {
      return;
    }

    yield;
  }
}
</script>
