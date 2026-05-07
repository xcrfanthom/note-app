<script setup lang="ts">
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
}>();

defineEmits<{
  (e: 'select-tab', index: number): void;
  (e: 'close-tab', index: number): void;
}>();
</script>

<template>
  <div class="tabs-bar" v-if="tabs.length > 0">
    <div
      v-for="(tab, idx) in tabs"
      :key="tab.path"
      class="tab"
      :class="{ active: idx === activeTabIndex }"
      @click="$emit('select-tab', idx)"
    >
      <span class="tab-name">{{ tab.name }}</span>
      <span v-if="tab.isDirty" class="dirty-indicator">●</span>
      <button class="tab-close" @click.stop="$emit('close-tab', idx)">×</button>
    </div>
  </div>
</template>

<style scoped>
.tabs-bar {
  display: flex;
  background: #2d2d2d;
  border-bottom: 1px solid #3c3c3c;
  overflow-x: auto;
  white-space: nowrap;
}

.tab {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 6px 12px;
  font-size: 12px;
  cursor: pointer;
  border-right: 1px solid #3c3c3c;
  background: #252526;
  color: #aaa;
  min-width: 80px;
  max-width: 180px;
  overflow: hidden;
  text-overflow: ellipsis;
}

.tab.active {
  background: #1e1e1e;
  color: #fff;
}

.tab-name {
  overflow: hidden;
  text-overflow: ellipsis;
  flex: 1;
}

.dirty-indicator {
  color: #e2b714;
  font-size: 14px;
}

.tab-close {
  background: transparent;
  border: none;
  color: #888;
  font-size: 14px;
  cursor: pointer;
  padding: 0 2px;
}

.tab-close:hover {
  color: #fff;
  background: #444;
  border-radius: 2px;
}
</style>