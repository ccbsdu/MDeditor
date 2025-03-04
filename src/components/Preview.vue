<template>
  <div class="preview markdown-body" :class="{ 'dark-theme': isDarkTheme }">
    <div v-html="renderedContent"></div>
  </div>
</template>

<script setup>
import { computed, ref, nextTick } from 'vue'
import MarkdownIt from 'markdown-it'
import mermaid from 'mermaid'

const props = defineProps({
  content: {
    type: String,
    default: ''
  }
})

const isDarkTheme = ref(false)

// 修改 mermaid 初始化配置
mermaid.initialize({
  startOnLoad: false,  // 改为 false，由我们手动控制渲染
  theme: 'default',
  securityLevel: 'loose',
  fontSize: 16
})

// 更新渲染函数
const renderMermaid = async () => {
  try {
    // 清除之前的渲染结果
    document.querySelectorAll('.mermaid').forEach(node => {
      node.removeAttribute('data-processed')
    })
    // 重新渲染所有 mermaid 图表
    await mermaid.run()
  } catch (error) {
    console.error('Mermaid rendering error:', error)
  }
}

const previewContent = ref(null)

const md = new MarkdownIt({
  html: true,
  linkify: true,
  typographer: true
})

const renderedContent = computed(() => {
  const content = props.content
  const rendered = md.render(content)
  nextTick(() => {
    renderMermaid()
  })
  return rendered
})

const copyFormatted = async () => {
  try {
    await navigator.clipboard.writeText(previewContent.value.innerHTML)
    alert('已复制预览内容！')
  } catch (err) {
    console.error('复制失败:', err)
    alert('复制失败，请重试')
  }
}
</script>

<style>
/* 添加 mermaid 图表样式 */
.mermaid {
  margin: 1em 0;
  overflow: auto;
}
</style>

<style scoped>
.preview {
  height: 100%;
  padding: 20px;
  overflow: auto;
}

/* Set font size for paragraph elements only */
.preview .markdown-body p,
.preview .markdown-body li,
.preview .markdown-body td,
.preview .markdown-body th,
.preview .markdown-body blockquote p {
  font-size: 14px;
  line-height: 1.6;
}

.preview.dark-theme {
  background-color: #282c34;
  color: #abb2bf;
}

.preview.dark-theme .markdown-body {
  color: #abb2bf;
  background-color: #282c34;
}

.preview.dark-theme .markdown-body a {
  color: #61afef;
}

.preview.dark-theme .markdown-body hr {
  border-color: #3e4451;
}

.preview.dark-theme .markdown-body blockquote {
  border-left-color: #3e4451;
}
</style>