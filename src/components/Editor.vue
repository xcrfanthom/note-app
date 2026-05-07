<script setup lang="ts">
import TabsBar from './TabsBar.vue';

interface Tab {
  path: string;
  name: string;
  content: string;
  isDirty: boolean;
  type?: 'text' | 'rich';
}

defineProps<{
  tabs: Tab[];
  activeTabIndex: number;
  activeFilePath: string | null;
  content: string;
}>();

defineEmits<{
  (e: 'select-tab', index: number): void;
  (e: 'close-tab', index: number): void;
  (e: 'update:content', value: string): void;
  (e: 'save'): void;
}>();
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
      <button @click="$emit('save')" :disabled="!activeFilePath">
        Save Changes
      </button>
    </div>
    <textarea
      :value="content"
      @input="$emit('update:content', ($event.target as HTMLTextAreaElement).value)"
      spellcheck="false"
      placeholder="Start typing…"
      class="editor-textarea"
      :disabled="!activeFilePath"
    ></textarea>
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

.editor-header button:hover:not(:disabled) {
  background: #108ad1;
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
}

.editor-textarea:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}
</style>