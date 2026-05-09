<script setup lang="ts">
import { onUnmounted, computed, ref } from 'vue'
import TabsBar from './TabsBar.vue'
import { marked } from 'marked'
import DOMPurify from 'dompurify'

interface Tab {
  path: string;
  name: string;
  content: string;
  isDirty: boolean;
  type?: 'text' | 'rich' | 'markdown' | 'tex' | 'draw';
}

type Mode = 'code' | 'preview' | 'split'
const mode = ref<Mode>('code')

const props = defineProps<{
  tabs: Tab[];
  activeTabIndex: number;
  activeFilePath: string | null;
  content: string;
}>()

defineEmits<{
  (e: 'select-tab', index: number): void;
  (e: 'close-tab', index: number): void;
  (e: 'update:content', value: string): void;
  (e: 'save'): void;
}>()

const renderedMarkdown = computed(() => {
  if (!props.content) return 'Nothing to preview...'
  return DOMPurify.sanitize(marked(props.content) as string)
})

const splitPercent = ref(50)
const isDragging = ref(false)
let isResizing = false

function startResize() {
  isResizing = true
  isDragging.value = true
  document.addEventListener('mousemove', onResize)
  document.addEventListener('mouseup', stopResize)
}

function onResize(e: MouseEvent) {
  if (!isResizing) return
  const container = document.querySelector('.split-area') as HTMLElement
  if (!container) return
  const rect = container.getBoundingClientRect()
  const percent = ((e.clientX - rect.left) / rect.width) * 100
  splitPercent.value = Math.min(Math.max(percent, 20), 80)
}

function stopResize() {
  isResizing = false
  isDragging.value = false
  document.removeEventListener('mousemove', onResize)
  document.removeEventListener('mouseup', stopResize)
}

onUnmounted(() => {
  document.removeEventListener('mousemove', onResize)
  document.removeEventListener('mouseup', stopResize)
})
</script>

<template>
  <main class="editor-area">
    <TabsBar
      :tabs="tabs"
      :active-tab-index="activeTabIndex"
      @select-tab="$emit('select-tab', $event)"
      @close-tab="$emit('close-tab', $event)"
    />
    <div v-if="tabs.length === 0" class="no-tabs-placeholder">
      No file open. Select a file from the left.
    </div>

    <div class="editor-header" v-if="activeFilePath">
      <input :value="activeFilePath" readonly placeholder="No file selected" />
      <div class="toolbar-buttons">
        <button :class="{ active: mode === 'code' }" @click="mode = 'code'">Code</button>
        <button :class="{ active: mode === 'preview' }" @click="mode = 'preview'">Preview</button>
        <button :class="{ active: mode === 'split' }" @click="mode = 'split'">Split</button>
      </div>
      <button @click="$emit('save')" :disabled="!activeFilePath">
        Save Changes
      </button>
    </div>
    
    <!-- Code view -->
    <textarea
      v-if="mode === 'code'"
      :value="content"
      @input="$emit('update:content', ($event.target as HTMLTextAreaElement).value)"
      spellcheck="false"
      placeholder="Start typing…"
      class="editor-textarea"
      :disabled="!activeFilePath"
    />

    <!-- Preview view -->
    <div v-else-if="mode === 'preview'" v-html="renderedMarkdown" class="markdown-preview" />

    <!-- Split view -->
    <div v-else-if="mode === 'split'" class="split-area" :class="{ dragging: isDragging }">
      <textarea
        :style="{ width: splitPercent + '%' }"
        :value="content"
        @input="$emit('update:content', ($event.target as HTMLTextAreaElement).value)"
        spellcheck="false"
        placeholder="Start typing…"
        class="editor-textarea"
        :disabled="!activeFilePath"
      />
      <div class="split-divider" @mousedown="startResize" />
      <div class="markdown-preview" :style="{ width: (100 - splitPercent) + '%' }" v-html="renderedMarkdown" />
    </div>
  </main>
</template>

<style scoped>
.editor-area {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-width: 0;
}

.no-tabs-placeholder {
  padding: 12px;
  color: #666;
  font-style: italic;
  border-bottom: 1px solid #3c3c3c;
}

.editor-header {
  padding: 10px;
  background: #2d2d2d;
  display: flex;
  gap: 10px;
  border-bottom: 1px solid #3c3c3c;
}

.editor-header input {
  flex: 1;
  background: #1e1e1e;
  border: 1px solid #3c3c3c;
  color: #888;
  padding: 6px 12px;
  font-size: 11px;
  border-radius: 4px;
}

.editor-header .toolbar-buttons {
  background: #007acc;
  color: white;
  border: none;
  padding: 6px 15px;
  border-radius: 4px;
  font-size: 12px;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s;
}

.toolbar-buttons button.active {
  background: #034566;
  color: white;
}

.toolbar-buttons button:hover {
  background: #108ad1;
}

.editor-header button {
  background: #007acc;
  color: white;
  border: none;
  padding: 6px 15px;
  border-radius: 4px;
  font-size: 12px;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s;
}

.editor-header button:disabled {
  background: #3a3a3a;
  color: #777;
  cursor: not-allowed;
}

.editor-textarea {
  flex: 1;
  background: #1e1e1e;
  color: #d4d4d4;
  border: none;
  padding: 20px;
  font-family: 'Consolas', 'Monaco', 'Courier New', monospace;
  font-size: 14px;
  line-height: 1.6;
  resize: none;
  outline: none;
  box-sizing: border-box;
}

.editor-textarea:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.split-area {
  flex: 1;
  display: flex;
  overflow: hidden;
}

.split-area.dragging {
  user-select: none;
  cursor: col-resize;
}

.split-pane {
  overflow: auto;
  flex-shrink: 0;
}

.split-divider {
  width: 4px;
  background: #3c3c3c;
  cursor: col-resize;
  flex-shrink: 0;
  transition: background 0.15s;
}

.markdown-preview {
  padding: 20px;
  color: #d4d4d4;
  background: #252526;
  overflow-y: auto;
  height: 100%;
  box-sizing: border-box;
}

.markdown-preview :deep(h1),
.markdown-preview :deep(h2),
.markdown-preview :deep(h3) {
  color: #fff;
  margin-top: 1.2em;
  margin-bottom: 0.4em;
  border-bottom: 1px solid #3c3c3c;
  padding-bottom: 4px;
}

.markdown-preview :deep(p) {
  margin: 0.6em 0;
  line-height: 1.7;
}

.markdown-preview :deep(code) {
  background: #2d2d2d;
  padding: 2px 6px;
  border-radius: 3px;
  font-family: 'Consolas', monospace;
  font-size: 13px;
  color: #f6f6f6;
}

.markdown-preview :deep(pre) {
  background: #2d2d2d;
  padding: 12px 16px;
  border-radius: 6px;
  overflow-x: auto;
}

.markdown-preview :deep(pre code) {
  background: none;
  padding: 0;
}

.markdown-preview :deep(blockquote) {
  border-left: 3px solid #007acc;
  margin: 0;
  padding-left: 16px;
  color: #888;
}

.markdown-preview :deep(a) {
  color: #007acc;
}

.markdown-preview :deep(ul),
.markdown-preview :deep(ol) {
  padding-left: 24px;
}

.markdown-preview :deep(table) {
  border-collapse: collapse;
  width: 100%;
}

.markdown-preview :deep(th),
.markdown-preview :deep(td) {
  border: 1px solid #3c3c3c;
  padding: 6px 12px;
}

.markdown-preview :deep(th) {
  background: #2d2d2d;
}
</style>