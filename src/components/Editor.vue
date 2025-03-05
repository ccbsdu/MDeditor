<template>
  <div class="editor-container">
    <textarea
      v-model="content"
      class="editor"
      @input="handleInput"
    ></textarea>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  modelValue: {
    type: String,
    default: ''
  }
})

const emit = defineEmits(['update:modelValue'])
const content = ref(props.modelValue)

const handleInput = (e) => {
  emit('update:modelValue', e.target.value)
}
</script>

<style scoped>
.editor-container {
  height: 100%;
  display: flex;
  flex-direction: column;
}

.editor {
  flex: 1;
  width: 100%;
  padding: 10px;
  border: none;
  resize: none;
  outline: none;
  font-size: 14px;
  line-height: 1.6;
  white-space: pre-wrap;
  word-wrap: break-word;
  background-color: #fff;
}
</style>