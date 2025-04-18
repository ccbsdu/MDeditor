<template>
  <div class="app-container">
    <div class="editor-preview-container">
      <Editor v-model="content" class="editor-pane" />
      <!-- 修改：预览区域的包裹容器 -->
      <div class="preview-wrapper">
        <!-- 修改：预览窗口的头部/工具栏 -->
        <div class="preview-toolbar">
          <button @click="copyToClipboard">复制预览结果</button>
          <!-- 这里可以添加其他预览相关的按钮 -->
        </div>
        <!-- 修改：预览内容区域 -->
        <div class="preview-pane markdown-body" ref="preview">
          <!-- Markdown 内容将通过 innerHTML 注入这里 -->
        </div>
      </div>
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

// 定义默认内容
const defaultContent = `# 欢迎使用 Markdown 编辑器\n\n` +
        `## 基本语法示例\n\n` +
        `### 文本格式\n` +
        `这是一个 **粗体** 和 *斜体* 的示例。\n\n` +
        `### 列表\n` +
        `1. 有序列表项 1\n` +
        `2. 有序列表项 2\n` +
        `   - 无序子列表\n` +
        `   - 另一个子项\n\n` +
        `### 引用\n` +
        `> 这是一段引用文本\n\n` +
        `### 代码\n` +
        '```javascript\n' +
        'console.log(\'Hello, Markdown!\');\n' +
        '```\n\n' +
        `### 表格\n` +
        `| 标题 1 | 标题 2 |\n` +
        `|--------|--------|\n` +
        `| 单元格 1 | 单元格 2 |\n\n` +
        `## LaTeX 公式示例\n\n` +
        `行内公式：$E = mc^2$ 和 $\\sqrt{x^2 + y^2}$\n\n` +
        `独立公式：\n\n` +
        `$$\n` +
        `\\int_{-\\infty}^{\\infty} e^{-x^2} dx = \\sqrt{\\pi}\n` +
        `$$\n\n` +
        `多行公式：\n\n` +
        `$$\n` +
        `\\begin{aligned}\n` +
        `f(x) &= \\int_{-\\infty}^{\\infty} \\hat{f}(\\xi) e^{2\\pi i \\xi x} d\\xi \\\\\n` +
        `\\hat{f}(\\xi) &= \\int_{-\\infty}^{\\infty} f(x) e^{-2\\pi i \\xi x} dx\n` +
        `\\end{aligned}\n` +
        `$$\n\n` +
        `## Mermaid 流程图示例\n\n` +
        '```mermaid\n' +
        'graph TD\n' +
        '    A[开始] --> B{条件判断}\n' +
        '    B -- 是 --> C[执行操作1]\n' +
        '    B -- 否 --> D[执行操作2]\n' +
        '    C --> E[结束]\n' +
        '    D --> E\n' +
        '```\n\n' +
        `## Mermaid 时序图示例\n\n` +
        '```mermaid\n' +
        'sequenceDiagram\n' +
        '    participant Alice\n' +
        '    participant Bob\n' +
        '    Alice->>Bob: 你好，Bob！\n' +
        '    Bob-->>Alice: 你好，Alice！\n' +
        '```';

export default {
  name: 'App',
  components: {
    Editor
  },
  data() {
    return {
      // 确认使用 defaultContent 初始化 content
      content: defaultContent 
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
      immediate: true // 确保初始加载时也执行渲染
    }
  },
  methods: {
    async renderMermaid() {
      const elements = this.$refs.preview.getElementsByClassName('mermaid')
      for (let element of elements) {
        if (!element.hasAttribute('data-processed')) {
          try {
            const id = `mermaid-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`
            const { svg } = await mermaid.render(id, element.textContent);
            element.innerHTML = svg
            element.setAttribute('data-processed', 'true')
          } catch (error) {
            console.error('Mermaid rendering failed:', error)
          }
        }
      }
    },
    async copyToClipboard() {
      try {
        const htmlContent = this.$refs.preview.innerHTML;
        const textContent = this.$refs.preview.innerText; // 作为备选的纯文本

        // 创建包含 HTML 和纯文本的 Blob
        const htmlBlob = new Blob([htmlContent], { type: 'text/html' });
        const textBlob = new Blob([textContent], { type: 'text/plain' });

        // 创建 ClipboardItem
        const clipboardItem = new ClipboardItem({
          'text/html': htmlBlob,
          'text/plain': textBlob
        });

        // 写入剪贴板
        await navigator.clipboard.write([clipboardItem]);

        alert('预览结果已作为富文本复制到剪贴板！');
      } catch (error) {
        console.error('复制到剪贴板失败:', error);
        // 备选方案：如果 Clipboard API 失败或不可用，尝试旧方法复制纯文本
        try {
          const el = document.createElement('textarea');
          el.value = this.$refs.preview.innerText; // 复制纯文本
          el.setAttribute('readonly', '');
          el.style.position = 'absolute';
          el.style.left = '-9999px';
          document.body.appendChild(el);
          el.select();
          document.execCommand('copy');
          document.body.removeChild(el);
          alert('已复制纯文本到剪贴板（富文本复制失败）');
        } catch (fallbackError) {
          console.error('备选复制方法也失败:', fallbackError);
          alert('复制失败，请手动复制。');
        }
      }
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

/* 修改：preview-wrapper 样式 */
.preview-wrapper {
  /* position: relative; */ /* 不再需要相对定位 */
  flex: 1;
  height: 100%;
  display: flex;
  flex-direction: column; /* 垂直排列：工具栏在上，内容在下 */
  border: 1px solid #ddd;
  border-radius: 4px;
  background-color: #fff;
  overflow: hidden; /* 防止内容溢出边框 */
}

/* 新增/修改：preview-toolbar 样式，使其类似编辑器的工具栏 */
.preview-toolbar {
  padding: 8px;
  border-bottom: 1px solid #ddd;
  background-color: #f5f5f5; /* 与编辑器工具栏背景色一致 */
  display: flex; /* 让按钮可以水平排列 */
  gap: 8px;
  flex-shrink: 0; /* 防止工具栏被压缩 */
}

.preview-toolbar button {
  padding: 4px 8px; /* 调整按钮大小 */
  background-color: #4a9eff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

/* 移除旧的 preview-header 样式 */
/*
.preview-header { ... }
*/

/* 修改：调整 preview-pane 样式 */
.preview-pane {
  flex: 1; /* 占据剩余空间 */
  overflow-y: auto; /* 仅内容区域滚动 */
  padding: 20px; /* 保留内容内边距 */
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
