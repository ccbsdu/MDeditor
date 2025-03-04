<template>
  <div class="container">
    <header>
      <h1>Markdown 编辑器</h1>
      <div class="actions">
        <button @click="saveContent">保存</button>
        <button @click="loadContent">打开</button>
        <button @click="clearContent" class="clear-btn">清空</button>
      </div>
    </header>
    <div class="editor-container">
      <div class="editor-pane">
        <Editor v-model="content" />
      </div>
      <div class="preview-pane">
        <div class="preview-header">
          <button class="copy-btn" @click="copyPreview">复制预览内容</button>
        </div>
        <Preview :content="content" ref="previewRef" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import Editor from './components/Editor.vue'
import Preview from './components/Preview.vue'

const content = ref(`# 欢迎使用 Markdown 编辑器

## 基本语法示例

### 文本格式
这是一个 **粗体** 和 *斜体* 的示例。

### 列表
1. 有序列表项 1
2. 有序列表项 2
   - 无序子列表
   - 另一个子项

### 引用
> 这是一段引用文本

### 代码
\`\`\`javascript
console.log('Hello, Markdown!');
\`\`\`

### 表格
| 标题 1 | 标题 2 |
|--------|--------|
| 单元格 1 | 单元格 2 |
`)

const saveContent = () => {
  const blob = new Blob([content.value], { type: 'text/markdown' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = 'document.md'
  a.click()
  URL.revokeObjectURL(url)
}

const loadContent = () => {
  const input = document.createElement('input')
  input.type = 'file'
  input.accept = '.md'
  input.onchange = (e) => {
    const file = e.target.files[0]
    const reader = new FileReader()
    reader.onload = (e) => {
      content.value = e.target.result
    }
    reader.readAsText(file)
  }
  input.click()
}

const clearContent = () => {
  if (confirm('确定要清空编辑器内容吗？')) {
    content.value = `# `  // 保留一个标题符号，方便用户直接开始输入
  }
}

const previewRef = ref(null)

const copyPreview = async () => {
  try {
    // 创建一个临时容器来处理格式化的内容
    const tempDiv = document.createElement('div')
    tempDiv.innerHTML = document.querySelector('.markdown-body').innerHTML
    
    // 创建一个新的 ClipboardItem
    const blob = new Blob([tempDiv.innerHTML], { type: 'text/html' })
    const data = new ClipboardItem({
      'text/html': blob
    })
    
    // 使用新的 Clipboard API 复制富文本
    await navigator.clipboard.write([data])
    alert('已复制格式化内容！')
  } catch (err) {
    // 后备方法：使用传统的 execCommand
    const selection = window.getSelection()
    const range = document.createRange()
    range.selectNodeContents(document.querySelector('.markdown-body'))
    selection.removeAllRanges()
    selection.addRange(range)
    try {
      document.execCommand('copy')
      alert('已复制格式化内容！')
    } catch (err) {
      alert('复制失败，请重试')
    }
    selection.removeAllRanges()
  }
}
</script>

<style>
.container {
  height: 100vh;
  display: flex;
  flex-direction: column;
}

header {
  padding: 1rem;
  background: #f5f5f5;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.actions button {
  margin-left: 10px;
  padding: 8px 16px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.actions button:hover {
  background: #45a049;
}

.editor-container {
  flex: 1;
  display: flex;
}

.editor-pane, .preview-pane {
  flex: 1;
  overflow: hidden;
  padding: 1rem;
}

.editor-pane {
  border-right: 1px solid #ddd;
}

.preview-header {
  padding: 10px;
  border-bottom: 1px solid #eee;
}

.copy-btn {
  padding: 6px 12px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.copy-btn:hover {
  background: #45a049;
}

.clear-btn {
  background: #ff4444 !important;
}

.clear-btn:hover {
  background: #cc0000 !important;
}
</style>
