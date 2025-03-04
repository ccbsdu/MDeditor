<template>
  <div class="toolbar">
    <div class="tool-group">
      <button @click="insertText('**', '**')" title="加粗">B</button>
      <button @click="insertText('*', '*')" title="斜体">I</button>
      <button @click="insertText('# ')" title="标题">H</button>
      <button @click="insertList('- ')" title="无序列表">•</button>
      <button @click="insertList('1. ')" title="有序列表">1.</button>
    </div>
    <div class="tool-group">
      <button @click="insertMermaidTemplate('flowchart')" title="流程图">Flow</button>
      <button @click="insertMermaidTemplate('sequence')" title="时序图">Seq</button>
      <button @click="insertMermaidTemplate('gantt')" title="甘特图">Gantt</button>
    </div>
    <div class="tool-group">
      <select v-model="theme" @change="changeTheme">
        <option value="light">浅色主题</option>
        <option value="dark">深色主题</option>
      </select>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const emit = defineEmits(['insert', 'theme-change'])
const theme = ref('light')

const insertText = (prefix, suffix = '') => {
  emit('insert', { prefix, suffix })
}

const insertList = (prefix) => {
  emit('insert', { prefix, newline: true })
}

const insertMermaidTemplate = (type) => {
  const templates = {
    flowchart: '```mermaid\ngraph TD\n    A[开始] --> B{判断}\n    B -- Yes --> C[处理]\n    B -- No --> D[结束]\n    C --> D\n```\n',
    sequence: '```mermaid\nsequenceDiagram\n    participant A as 用户\n    participant B as 系统\n    A->>B: 请求数据\n    B-->>A: 返回数据\n```\n',
    gantt: '```mermaid\ngantt\n    title 项目计划\n    section 阶段1\n    任务1    :a1, 2023-01-01, 30d\n    任务2    :after a1, 20d\n```\n'
  }
  emit('insert', { text: templates[type] })
}

const changeTheme = () => {
  emit('theme-change', theme.value)
}
</script>

<style scoped>
.toolbar {
  padding: 8px;
  border-bottom: 1px solid #ddd;
  display: flex;
  gap: 16px;
}

.tool-group {
  display: flex;
  gap: 4px;
}

button {
  padding: 4px 8px;
  border: 1px solid #ddd;
  background: #fff;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background: #f5f5f5;
}

select {
  padding: 4px 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}
</style>