<template>
  <div class="app-container">
    <div class="editor-preview-container">
      <Editor v-model="content" class="editor-pane" />
      <div class="preview-pane markdown-body" ref="preview"></div>
    </div>
  </div>
</template>

<script>
import Editor from './components/editor.vue'
import { marked } from 'marked'
import katex from 'katex'
import mermaid from 'mermaid'
import 'katex/dist/katex.min.css'

// 配置 mermaid
// 修改 mermaid 配置
mermaid.initialize({
  startOnLoad: true,
  theme: 'default',
  securityLevel: 'loose'
})

// 修改 marked 配置
marked.setOptions({
  renderer: new marked.Renderer(),
  highlight: function(code, lang) {
    if (lang === 'mermaid') {
      return `<pre class="mermaid">${code}</pre>`
    }
    return code
  }
})

// 自定义数学公式渲染
const originalRenderer = new marked.Renderer()
const renderer = {
  paragraph(text) {
    const pattern = /\$\$([\s\S]*?)\$\$|\$([\s\S]*?)\$/g
    let result = text
    let match

    while ((match = pattern.exec(text)) !== null) {
      const isBlock = !!match[1]
      const formula = isBlock ? match[1] : match[2]
      const rendered = katex.renderToString(formula, {
        displayMode: isBlock,
        throwOnError: false
      })
      result = result.replace(match[0], rendered)
    }
    return originalRenderer.paragraph(result)
  }
}
marked.use({ renderer })

export default {
  name: 'App',
  components: {
    Editor
  },
  data() {
    return {
      content: ''
    }
  },
  watch: {
    content: {
      handler(val) {
        this.$nextTick(() => {
          if (this.$refs.preview) {
            this.$refs.preview.innerHTML = marked(val)
            this.renderMermaid()
          }
        })
      },
      immediate: true
    }
  },
  methods: {
    renderMermaid() {
      const elements = this.$refs.preview.getElementsByClassName('mermaid')
      Array.from(elements).forEach(element => {
        if (!element.hasAttribute('data-processed')) {
          try {
            const id = `mermaid-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`
            mermaid.render(id, element.textContent).then(({ svg }) => {
              element.innerHTML = svg
              element.setAttribute('data-processed', 'true')
            })
          } catch (error) {
            console.error('Mermaid rendering failed:', error)
          }
        }
      })
    }
  }
}
</script>

<style>
.app-container {
  height: 100vh;
  padding: 20px;
}

.editor-preview-container {
  display: flex;
  gap: 20px;
  height: 100%;
}

.editor-pane {
  flex: 1;
  height: 100%;
}

.preview-pane {
  flex: 1;
  height: 100%;
  overflow-y: auto;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 4px;
  background-color: #fff;
}

/* 添加一些基础的 Markdown 样式 */
.markdown-body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif;
  font-size: 16px;
  line-height: 1.6;
  word-wrap: break-word;
}

.markdown-body pre {
  background-color: #f6f8fa;
  border-radius: 3px;
  padding: 16px;
  overflow: auto;
}

.markdown-body code {
  background-color: rgba(27,31,35,0.05);
  border-radius: 3px;
  padding: 0.2em 0.4em;
  font-family: SFMono-Regular, Consolas, "Liberation Mono", Menlo, monospace;
}

.markdown-body img {
  max-width: 100%;
  box-sizing: border-box;
}

/* 添加 KaTeX 相关样式 */
.katex-display {
  overflow-x: auto;
  overflow-y: hidden;
  padding: 1em 0;
}

/* 添加 Mermaid 相关样式 */
.mermaid {
  text-align: center;
  margin: 1em 0;
}
</style>
