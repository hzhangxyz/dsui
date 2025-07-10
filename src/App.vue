<template>
  <div class="container">
    <div v-for="(item, index) in arrayData" class="item-container">
      <textarea
        class="text-area"
        :value="item.text"
        @blur="(event) => updateText(index, event.target.value)"
      ></textarea>
      <div class="info-grid" v-if="item.text !== ''">
        <div v-if="item.from !== 'engine'" class="empty-column"></div>
        <div v-if="item.from !== 'engine'" class="empty-column"></div>
        <div v-if="item.from === 'engine'" class="info-card parent-rule">
          <h3>Parent Rule</h3>
          <div class="info-content">{{ item.parent_rule }}</div>
        </div>
        <div v-if="item.from === 'engine'" class="info-card parent-fact">
          <h3>Parent Fact</h3>
          <div class="info-content">{{ item.parent_fact }}</div>
        </div>
        <div class="info-card basic-info">
          <h3>Basic Info</h3>
          <div class="info-item">is rule: {{ item.rule }}</div>
          <div class="info-item">is fact: {{ item.fact }}</div>
          <div class="info-item">round: {{ item.round }}</div>
          <div class="info-item">from: {{ item.from }}</div>
        </div>
      </div>
    </div>
  </div>

  <div class="global-actions">
    <button class="action-button init-data" @click="initData">
      Clear Data
    </button>
    <button class="action-button load-data" @click="loadData">Load Data</button>
    <button class="action-button dump-data" @click="dumpData">Dump Data</button>
    <button class="action-button update-data" @click="updateDataset">
      Update Data
    </button>
  </div>
</template>

<style scoped>
.init-data {
  bottom: 240px;
  background: linear-gradient(135deg, #ff8a00, #ff5000);
}

.load-data {
  bottom: 180px;
  background: linear-gradient(135deg, #ffc371, #ff5f6d);
}

.dump-data {
  bottom: 120px;
  background: linear-gradient(135deg, #4facfe, #00f2fe);
}

.update-data {
  bottom: 60px;
  background: linear-gradient(135deg, #6e8efb, #a777e3);
}

.action-button {
  position: fixed;
  width: 150px;
  right: 30px;
  padding: 12px 28px;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  z-index: 100;
  font-size: 16px;
  font-weight: 500;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
}

.update-button:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.15);
}

.container {
  margin-top: 20px;
  padding: 20px;
  margin-bottom: 100px;
}

.item-container {
  display: flex;
  margin-bottom: 40px;
  align-items: flex-start;
  gap: 30px;
}

.text-area {
  height: 140px;
  width: 400px;
  padding: 15px;
  font-size: 15px;
  line-height: 1.6;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  resize: none;
  background-color: #fafafa;
  transition: border 0.3s ease;
}

.text-area:focus {
  outline: none;
  border-color: #6e8efb;
  box-shadow: 0 0 0 2px rgba(110, 142, 251, 0.2);
}

.info-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
  flex-grow: 1;
}

.info-card {
  padding: 18px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.basic-info {
  background: linear-gradient(135deg, #f5f7fa 0%, #e4e8eb 100%);
  border-left: 4px solid #6e8efb;
}

.parent-rule {
  background: linear-gradient(135deg, #f0f9ff 0%, #e0f2fe 100%);
  border-left: 4px solid #4facfe;
}

.parent-fact {
  background: linear-gradient(135deg, #f6f0ff 0%, #e9d8fd 100%);
  border-left: 4px solid #a779e3;
}

.info-card h3 {
  margin-top: 0;
  margin-bottom: 15px;
  font-size: 15px;
  color: #444;
  font-weight: 600;
}

.info-item,
.info-content {
  margin-bottom: 10px;
  font-size: 14px;
  color: #555;
  line-height: 1.5;
  word-break: break-word;
}

.info-content {
  white-space: pre-wrap;
}
</style>

<script setup>
import { ref } from "vue";
import { rule_t } from "./jsds.mjs";

let arrayData = ref([{}]);
let round = 1;
let lastRound = 0;

const updateText = (index, input) => {
  if (input === "") {
    if (index !== arrayData.value.length - 1) {
      arrayData.value.splice(index, 1);
    }
    return;
  }
  if (input === arrayData.value[index].text) {
    return;
  }
  const elem = new rule_t(input);
  arrayData.value[index] = {
    text: elem.toString(),
    rule: elem.length() !== 0,
    fact: elem.length() === 0,
    round: round++,
    from: "user",
  };
  if (index === arrayData.value.length - 1) {
    arrayData.value.push({});
  }
};

const updateDataset = () => {
  let length = arrayData.value.length - 1;
  let pool = new Set();
  for (let i = 0; i < length; ++i) {
    let text = arrayData.value[i].text;
    pool.add(text);
  }
  for (let i = 0; i < length; ++i) {
    let rule = arrayData.value[i];
    if (rule.text === "") {
      continue;
    }
    if (rule.fact) {
      continue;
    }
    for (let j = 0; j < length; ++j) {
      let fact = arrayData.value[j];
      if (fact.text === "") {
        continue;
      }
      if (fact.rule) {
        continue;
      }
      if (rule.round <= lastRound && fact.round <= lastRound) {
        continue;
      }
      const elem = new rule_t(rule.text).match(new rule_t(fact.text));
      if (elem === null) {
        continue;
      }
      const text = elem.toString();
      if (pool.has(text)) {
        continue;
      } else {
        pool.add(text);
      }
      let item = {
        text: text,
        rule: elem.length() !== 0,
        fact: elem.length() === 0,
        round: round + 1,
        from: "engine",
        parent_rule: rule.text,
        parent_fact: fact.text,
      };
      arrayData.value[arrayData.value.length - 1] = item;
      arrayData.value.push({});
    }
  }
  lastRound = round++;
};

const dumpData = () => {
  const result = {
    round: round,
    lastRound: lastRound,
    dataArray: arrayData.value,
  };
  const text = JSON.stringify(result);
  const blob = new Blob([text], { type: "application/json" });
  const a = document.createElement("a");
  a.href = URL.createObjectURL(blob);
  a.download = "data.json";
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  URL.revokeObjectURL(a.href);
};

const loadData = (data) => {
  const input = document.createElement("input");
  input.type = "file";
  input.accept = "application/json";

  input.onchange = (event) => {
    const file = event.target.files[0];
    if (!file) {
      return;
    }

    const reader = new FileReader();
    reader.onload = (event) => {
      let text = event.target.result;
      let data;
      try {
        data = JSON.parse(text);
      } catch (error) {
        return;
      }
      round = data.round;
      lastRound = data.lastRound;
      arrayData.value = data.dataArray;
      if (
        arrayData.value.length === 0 ||
        arrayData.value[arrayData.value.length - 1]?.text === ""
      ) {
        arrayData.value.push({});
      }
    };
    reader.readAsText(file);
  };

  input.click();
};

const initData = () => {
  arrayData.value = [{}];
  updateText(0, "(`a < `b) (`b < `c) (`a < `c)");
  updateText(1, "(1 < 2)");
  updateText(2, "(2 < 3)");
};

initData();
</script>
