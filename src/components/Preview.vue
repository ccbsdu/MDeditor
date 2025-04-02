<template>
  <div class="preview-container">
    <div class="preview-toolbar">
      <button @click="copyFormatted" class="copy-btn">复制内容</button>
    </div>
    <div class="preview-content" ref="previewContent" v-html="renderedContent"></div>
  </div>
</template>

<style>
/* 保持原有样式不变 */
.preview-container {
  height: 100%;
  overflow: auto;
  padding: 10px 20px;
  background-color: #fff;
}

.preview-content {
  max-width: 100%;
  word-wrap: break-word;
}

/* 统一段落文本大小 */
.preview-content p {
  font-size: 14px;
  line-height: 1.6;
  margin: 1em 0;
}

/* 代码块样式 */
.preview-content pre {
  background-color: #f5f5f5;
  padding: 10px;
  border-radius: 4px;
  overflow-x: auto;
}

.preview-content code {
  font-family: Consolas, Monaco, 'Andale Mono', monospace;
  font-size: 14px;
}

/* 表格样式 */
.preview-content table {
  border-collapse: collapse;
  width: 100%;
  margin: 1em 0;
}

.preview-content th,
.preview-content td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

.preview-content th {
  background-color: #f5f5f5;
}

/* 引用样式 */
.preview-content blockquote {
  margin: 1em 0;
  padding-left: 1em;
  border-left: 4px solid #ddd;
  color: #666;
}

/* 列表样式 */
.preview-content ul,
.preview-content ol {
  padding-left: 2em;
  margin: 1em 0;
}

/* 链接样式 */
.preview-content a {
  color: #4a9eff;
  text-decoration: none;
}

.preview-content a:hover {
  text-decoration: underline;
}

/* 图片样式 */
.preview-content img {
  max-width: 100%;
  height: auto;
  display: block;
  margin: 1em auto;
}

/* KaTeX 公式样式 */
.katex {
  font-size: 1.1em !important;
}

.katex-display {
  margin: 1em 0 !important;
  overflow-x: auto;
  overflow-y: hidden;
}

/* texmath 插件生成的公式容器样式 */
.texmath {
  text-align: center;
  margin: 1em 0;
}

.error {
  color: red;
  background-color: #ffeeee;
  padding: 10px;
  border-radius: 4px;
  margin: 10px 0;
}

/* 添加工具栏样式 */
.preview-toolbar {
  padding: 8px;
  border-bottom: 1px solid #ddd;
  background-color: #f5f5f5;
}

.copy-btn {
  padding: 6px 12px;
  background-color: #4a9eff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.copy-btn:hover {
  background-color: #3a8eef;
}
</style>

<script>
import MarkdownIt from 'markdown-it'
import texmath from 'markdown-it-texmath'
import katex from 'katex'
import 'katex/dist/katex.min.css'

export default {
  name: 'Preview',
  props: {
    content: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      md: null
    }
  },
  created() {
    // 创建 markdown-it 实例并配置
    this.md = new MarkdownIt({
      html: true,
      breaks: true,
      linkify: true
    });
    
    // 使用 texmath 插件，配置 katex 作为渲染引擎
    this.md.use(texmath, {
      engine: katex,
      delimiters: 'dollars',  // 使用 $...$ 和 $$...$$ 作为分隔符
      katexOptions: {
        macros: {
          "\\RR": "\\mathbb{R}"
        },
        throwOnError: false,
        strict: false
      }
    });
  },
  computed: {
    renderedContent() {
      if (!this.content) {
        return '';
      }
      
      try {
        // 直接使用 markdown-it 渲染，插件会自动处理 LaTeX 公式
        let html = this.md.render(this.content);
        return html;
      } catch (e) {
        console.error('渲染错误:', e);
        return `<div class="error">渲染错误: ${e.message}</div>`;
      }
    }
  },
  methods: {
    copyFormatted() {
      if (!this.$refs.previewContent) return
      
      try {
        // 创建一个临时容器
        const tempDiv = document.createElement('div');
        
        // 获取 KaTeX CSS
        const katexCss = document.querySelector('link[href*="katex"]');
        const katexStyles = katexCss ? `<link rel="stylesheet" href="${katexCss.href}">` : '';
        
        // 复制所有需要的样式表
        const styleSheets = Array.from(document.styleSheets);
        let styles = '';
        styleSheets.forEach(sheet => {
          try {
            if (!sheet.href || sheet.href.includes('katex')) return; // 跳过 KaTeX 样式表
            Array.from(sheet.cssRules).forEach(rule => {
              styles += rule.cssText + '\n';
            });
          } catch (e) {
            // 跨域样式表会报错，忽略
          }
        });
        
        // 组合完整的 HTML，包含 KaTeX 资源
        const fullHtml = `
          <!DOCTYPE html>
          <html>
            <head>
              ${katexStyles}
              <style>${styles}</style>
            </head>
            <body>
              <div class="preview-content">
                ${this.$refs.previewContent.innerHTML}
              </div>
            </body>
          </html>
        `;
        
        // 创建富文本数据
        const htmlBlob = new Blob([fullHtml], { type: 'text/html;charset=utf-8' });
        const textBlob = new Blob([this.$refs.previewContent.innerText], { type: 'text/plain;charset=utf-8' });
        
        // 使用新的 Clipboard API
        if (navigator.clipboard?.write) {
          navigator.clipboard.write([
            new ClipboardItem({
              'text/html': htmlBlob,
              'text/plain': textBlob
            })
          ]).then(() => {
            alert('内容已复制到剪贴板');
          }).catch(err => {
            console.error('复制失败:', err);
            this.fallbackCopyHtml();
          });
        } else {
          this.fallbackCopyHtml();
        }
      } catch (err) {
        console.error('复制失败:', err);
        this.fallbackCopyHtml();
      }
    },
    
    // 回退的富文本复制方法
    fallbackCopyHtml() {
      const selection = window.getSelection();
      const range = document.createRange();
      range.selectNodeContents(this.$refs.previewContent);
      selection.removeAllRanges();
      selection.addRange(range);
      
      try {
        document.execCommand('copy');
        alert('内容已复制到剪贴板');
      } catch (err) {
        alert('复制失败，请手动复制');
        console.error('复制失败:', err);
      } finally {
        selection.removeAllRanges();
      }
    }
  }
}
</script>