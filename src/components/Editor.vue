<template>
  <div class="editor-container">
    <div class="toolbar">
      <!-- 字体大小选择器 -->
      <select v-model="fontSize" class="font-size-select">
        <option value="12px">12px</option>
        <option value="14px">14px</option>
        <option value="16px">16px</option>
        <option value="18px">18px</option>
      </select>
      
      <!-- 格式按钮 -->
      <button @click="insertBold" class="format-btn">加粗</button>
      <button @click="insertItalic" class="format-btn">斜体</button>
      <button @click="insertLink" class="format-btn">链接</button>
      <button @click="insertImage" class="format-btn">图片</button>
      <button @click="insertCode" class="format-btn">代码</button>
      <!-- 新增：清空按钮 -->
      <button @click="clearContent" class="clear-button">清空</button>
      <div class="spacer"></div>
      <input type="text" v-model="searchText" placeholder="搜索文本" />
      <input type="text" v-model="replaceText" placeholder="替换为" />
      <button @click="searchAndHighlight">搜索</button>
      <button @click="replaceNext">替换</button>
      <button @click="replaceAll">全部替换</button>
    </div>
    <textarea
      ref="textarea"
      :value="modelValue"
      @input="updateValue($event.target.value)"
      @keydown.tab.prevent="handleTab"
      :style="{ fontSize: fontSize + 'px' }"
    ></textarea>
  </div>
</template>

<script>
export default {
  name: 'Editor',
  props: {
    modelValue: String,
  },
  emits: ['update:modelValue'],
  data() {
    return {
      fontSize: '14px', // 保留 fontSize
      searchText: '',
      replaceText: '',
      currentSearchIndex: 0,
      searchResults: [],
      lastSearchText: '',
      // defaultContent: ... // 删除这里的 defaultContent 定义
    }
  },
  methods: {
    updateValue(value) {
      this.$emit('update:modelValue', value);
    },
    
    // 插入格式方法
    insertBold() {
      this.insertText('**加粗文本**', 2, 6);
    },
    insertItalic() {
      this.insertText('*斜体文本*', 1, 5);
    },
    insertLink() {
      this.insertText('[链接文字](https://)', 1, 5);
    },
    insertImage() {
      this.insertText('![图片描述](图片URL)', 2, 6);
    },
    insertCode() {
      this.insertText('```\n代码块\n```', 3, 3);
    },
    insertText(text, selectionStart, selectionEnd) {
      const editor = this.$refs.editorEl;
      const startPos = editor.selectionStart;
      const endPos = editor.selectionEnd;
      const currentText = editor.value;
      
      editor.value = currentText.substring(0, startPos) + 
                    text + 
                    currentText.substring(endPos);
      
      editor.selectionStart = startPos + selectionStart;
      editor.selectionEnd = startPos + selectionEnd;
      editor.focus();
    },
    
    // 搜索和替换方法
    searchAndReplace(doReplace) {
      if (!this.searchText) return;
      
      const textarea = this.$refs.editorEl;
      const content = textarea.value;
      
      // 重置搜索结果
      if (this.searchText !== this.lastSearchText) {
        this.searchResults = [];
        this.currentSearchIndex = 0;
        
        // 查找所有匹配项
        let index = content.indexOf(this.searchText);
        while (index !== -1) {
          this.searchResults.push({
            from: index,
            to: index + this.searchText.length
          });
          index = content.indexOf(this.searchText, index + 1);
        }
        
        this.lastSearchText = this.searchText;
      }
      
      if (this.searchResults.length === 0) {
        alert('未找到匹配项');
        return;
      }
      
      // 获取当前匹配项
      const current = this.searchResults[this.currentSearchIndex];
      
      // 选中文本并滚动到视图
      textarea.focus();
      textarea.setSelectionRange(current.from, current.to);
      
      // 确保选中的文本在视图中可见
      const textBeforeSelection = content.substring(0, current.from);
      const lineCount = (textBeforeSelection.match(/\n/g) || []).length;
      const lineHeight = 20; // 估计的行高
      textarea.scrollTop = lineCount * lineHeight;
      
      // 如果需要替换
      if (doReplace) {
        // 获取当前选中的文本
        const beforeText = content.substring(0, current.from);
        const afterText = content.substring(current.to);
        
        // 替换文本
        const newContent = beforeText + this.replaceText + afterText;
        this.$emit('update:modelValue', newContent);
        
        // 更新搜索结果中的位置
        const lengthDiff = this.replaceText.length - this.searchText.length;
        for (let i = this.currentSearchIndex + 1; i < this.searchResults.length; i++) {
          this.searchResults[i].from += lengthDiff;
          this.searchResults[i].to += lengthDiff;
        }
      }
      
      // 移动到下一个匹配项
      this.currentSearchIndex = (this.currentSearchIndex + 1) % this.searchResults.length;
    }, // <- 确保这里有逗号
    replaceAll() {
      if (!this.searchText) return;
      
      const textarea = this.$refs.editorEl;
      const content = textarea.value;
      
      if (content.indexOf(this.searchText) === -1) {
        alert('未找到匹配项');
        return;
      }
      
      // 使用正则表达式全局替换
      const newContent = content.split(this.searchText).join(this.replaceText);
      
      // 更新编辑器内容
      this.$emit('update:modelValue', newContent);
      
      // 重置搜索状态
      this.searchResults = [];
      this.currentSearchIndex = 0;
      this.lastSearchText = '';
      
      // 显示替换完成消息
      const count = (content.match(new RegExp(this.searchText.replace(/[.*+?^${}()|[\]\\]/g, '\\$&'), 'g')) || []).length;
      alert(`已替换 ${count} 处匹配项`);
    }, // 在这里添加逗号
    
    // 新增：清空内容的方法
    clearContent() {
      this.updateValue(''); // 发出事件将内容更新为空字符串
    },
    handleTab(event) {
      this.insertText('**加粗文本**', 2, 6);
    },
    searchAndHighlight() {
      this.insertText('*斜体文本*', 1, 5);
    },
    replaceNext() {
      this.insertText('[链接文字](https://)', 1, 5);
    },
    replaceAll() {
      this.insertText('![图片描述](图片URL)', 2, 6);
    },
    insertCode() {
      this.insertText('```\n代码块\n```', 3, 3);
    },
    insertText(text, selectionStart, selectionEnd) {
      const editor = this.$refs.editorEl;
      const startPos = editor.selectionStart;
      const endPos = editor.selectionEnd;
      const currentText = editor.value;
      
      editor.value = currentText.substring(0, startPos) + 
                    text + 
                    currentText.substring(endPos);
      
      editor.selectionStart = startPos + selectionStart;
      editor.selectionEnd = startPos + selectionEnd;
      editor.focus();
    },
    
    // 搜索和替换方法
    searchAndReplace(doReplace) {
      if (!this.searchText) return;
      
      const textarea = this.$refs.editorEl;
      const content = textarea.value;
      
      // 重置搜索结果
      if (this.searchText !== this.lastSearchText) {
        this.searchResults = [];
        this.currentSearchIndex = 0;
        
        // 查找所有匹配项
        let index = content.indexOf(this.searchText);
        while (index !== -1) {
          this.searchResults.push({
            from: index,
            to: index + this.searchText.length
          });
          index = content.indexOf(this.searchText, index + 1);
        }
        
        this.lastSearchText = this.searchText;
      }
      
      if (this.searchResults.length === 0) {
        alert('未找到匹配项');
        return;
      }
      
      // 获取当前匹配项
      const current = this.searchResults[this.currentSearchIndex];
      
      // 选中文本并滚动到视图
      textarea.focus();
      textarea.setSelectionRange(current.from, current.to);
      
      // 确保选中的文本在视图中可见
      const textBeforeSelection = content.substring(0, current.from);
      const lineCount = (textBeforeSelection.match(/\n/g) || []).length;
      const lineHeight = 20; // 估计的行高
      textarea.scrollTop = lineCount * lineHeight;
      
      // 如果需要替换
      if (doReplace) {
        // 获取当前选中的文本
        const beforeText = content.substring(0, current.from);
        const afterText = content.substring(current.to);
        
        // 替换文本
        const newContent = beforeText + this.replaceText + afterText;
        this.$emit('update:modelValue', newContent);
        
        // 更新搜索结果中的位置
        const lengthDiff = this.replaceText.length - this.searchText.length;
        for (let i = this.currentSearchIndex + 1; i < this.searchResults.length; i++) {
          this.searchResults[i].from += lengthDiff;
          this.searchResults[i].to += lengthDiff;
        }
      }
      
      // 移动到下一个匹配项
      this.currentSearchIndex = (this.currentSearchIndex + 1) % this.searchResults.length;
    },
    
    replaceAll() {
      if (!this.searchText) return;
      
      const textarea = this.$refs.editorEl;
      const content = textarea.value;
      
      if (content.indexOf(this.searchText) === -1) {
        alert('未找到匹配项');
        return;
      }
      
      // 使用正则表达式全局替换
      const newContent = content.split(this.searchText).join(this.replaceText);
      
      // 更新编辑器内容
      this.$emit('update:modelValue', newContent);
      
      // 重置搜索状态
      this.searchResults = [];
      this.currentSearchIndex = 0;
      this.lastSearchText = '';
      
      // 显示替换完成消息
      const count = (content.match(new RegExp(this.searchText.replace(/[.*+?^${}()|[\]\\]/g, '\\$&'), 'g')) || []).length;
      alert(`已替换 ${count} 处匹配项`);
    }
  }
}
</script>

<style scoped>
/* 合并后的样式 */
.editor-container {
  height: 100%;
  display: flex;
  flex-direction: column;
}

.editor-toolbar {
  padding: 8px;
  border-bottom: 1px solid #ddd;
  background-color: #f5f5f5;
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.format-btn {
  padding: 4px 8px;
  background-color: #4a9eff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.font-size-select {
  padding: 4px;
  border-radius: 4px;
  border: 1px solid #ddd;
}

/* 修改：将 .editor 选择器改为 textarea */
textarea {
  flex: 1; /* 占据剩余空间 */
  width: 100%;
  padding: 10px;
  border: none; /* 移除这里的边框 */
  resize: none;
  outline: none;
  /* fontSize 应该由 :style 绑定控制，这里可以移除或设为默认值 */
  /* font-size: 14px; */ 
  line-height: 1.6;
  white-space: pre-wrap;
  word-wrap: break-word;
  background-color: #fff;
  font-family: monospace;
  /* margin-top: 8px; */ /* 可以移除，由父级 flex gap 控制 */
   /* 移除这里的边框和圆角，由 editor-container 控制 */
  /* border: 1px solid #ddd; */
  /* border-radius: 4px; */
}

.editor-container {
  height: 100%;
  display: flex;
  flex-direction: column;
  border: 1px solid #ddd; /* 将边框移到外层容器 */
  border-radius: 4px;   /* 将圆角移到外层容器 */
  overflow: hidden;       /* 隐藏内部溢出 */
}

.toolbar { /* 修改：确保工具栏样式类名正确 */
  padding: 8px;
  border-bottom: 1px solid #ddd;
  background-color: #f5f5f5;
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  flex-shrink: 0; /* 防止工具栏被压缩 */
}

.search-replace-container {
  display: flex;
  flex: 1;
  gap: 8px;
  margin-left: auto;
}

.search-replace-inputs {
  display: flex;
  gap: 8px;
}

.search-replace-inputs input {
  flex: 1;
  padding: 6px 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

.search-replace-buttons {
  display: flex;
  gap: 8px;
}

.search-replace-buttons button {
  padding: 6px 12px;
  background-color: #4a9eff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

@media (max-width: 768px) {
  .editor-toolbar {
    flex-direction: column;
  }
  
  .search-replace-container {
    margin-left: 0;
    width: 100%;
  }
}
.toolbar button.clear-button {
  background-color: #f44336; /* 红色背景以示警告 */
  color: white;
}

.toolbar button.clear-button:hover {
  background-color: #d32f2f;
}
</style>