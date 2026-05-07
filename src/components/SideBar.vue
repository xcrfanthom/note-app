<script setup lang="ts">
import { ref, computed } from 'vue';

interface FileItem {
  name: string;
  path: string;
  isDirectory: boolean;
}

defineProps<{
  files: FileItem[];
  loading: boolean;
  error: string;
  activeFilePath: string | null;
}>();

defineEmits<{
  (e: 'file-click', file: FileItem): void;
  (e: 'create-file'): void;
}>();

const collapsed = ref(false);
const width = ref(260);
const isResizing = ref(false);

const sidebarStyle = computed(() => ({
  width: collapsed.value ? '0px' : `${width.value}px`,
  minWidth: collapsed.value ? '0px' : `${width.value}px`,
  transition: isResizing.value ? 'none' : 'width 0.2s ease, min-width 0.2s ease',
}));

function startResize(e: MouseEvent) {
  e.preventDefault();
  isResizing.value = true;
  const startX = e.clientX;
  const startWidth = width.value;

  const onMouseMove = (ev: MouseEvent) => {
    ev.preventDefault();
    const delta = ev.clientX - startX;
    width.value = Math.max(160, Math.min(500, startWidth + delta));
  };

  const onMouseUp = () => {
    isResizing.value = false;
    document.removeEventListener('mousemove', onMouseMove);
    document.removeEventListener('mouseup', onMouseUp);
  };

  document.addEventListener('mousemove', onMouseMove);
  document.addEventListener('mouseup', onMouseUp);
}

function toggleCollapse() {
  collapsed.value = !collapsed.value;
}
</script>

<template>
  <aside
    class="browser-sidebar"
    :class="{ resizing: isResizing }"
    :style="sidebarStyle"
    v-show="!collapsed"
  >
    <div class="toolbar">
      <button @click="$emit('create-file')" class="new-file-btn" title="New text file">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <line x1="12" y1="5" x2="12" y2="19"></line>
          <line x1="5" y1="12" x2="19" y2="12"></line>
        </svg>
        <span class="btn-label">New .txt</span>
      </button>
      <button @click="toggleCollapse" class="collapse-btn" title="Collapse sidebar">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <polyline points="15 18 9 12 15 6"></polyline>
        </svg>
      </button>
    </div>
    <div class="sidebar-status">
      <div v-if="loading" class="status-text">Loading…</div>
      <div v-if="error" class="error-message">{{ error }}</div>
    </div>
    <ul class="file-list">
      <li
        v-for="file in files"
        :key="file.path"
        @click="$emit('file-click', file)"
        :class="{ active: file.path === activeFilePath }"
      >
        <span class="icon">
          <svg v-if="file.isDirectory" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M22 19a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h5l2 3h9a2 2 0 0 1 2 2z"></path>
          </svg>
          <svg v-else width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M13 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V9z"></path>
            <polyline points="13 2 13 9 20 9"></polyline>
            <line x1="16" y1="13" x2="8" y2="13"></line>
            <line x1="16" y1="17" x2="8" y2="17"></line>
          </svg>
        </span>
        <span class="file-name">{{ file.name }}</span>
      </li>
      <li v-if="files.length === 0 && !loading && !error" class="empty">
        Empty folder
      </li>
    </ul>
    <div class="resize-handle" @mousedown="startResize"></div>
  </aside>
  <div v-show="collapsed" class="collapsed-strip">
    <button @click="toggleCollapse" class="expand-btn" title="Expand sidebar">
      <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <polyline points="9 18 15 12 9 6"></polyline>
      </svg>
    </button>
  </div>
</template>

<style scoped>
.browser-sidebar {
  border-right: 1px solid #333;
  background: #252526;
  display: flex;
  flex-direction: column;
  position: relative;
  overflow: hidden;
}

.browser-sidebar * {
  max-width: 100%;
  overflow-x: hidden;
}

.toolbar {
  padding: 8px 12px;
  border-bottom: 1px solid #3c3c3c;
  background: #2d2d2d;
  display: flex;
  gap: 6px;
  align-items: center;
  flex-shrink: 0;
}

.new-file-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  background: #0e639c;
  color: white;
  border: none;
  padding: 6px 12px;
  border-radius: 4px;
  font-size: 12px;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s;
  flex: 1;
}

.new-file-btn:hover {
  background: #1177bb;
}

.collapse-btn {
  background: transparent;
  border: none;
  color: #ccc;
  cursor: pointer;
  padding: 4px;
  border-radius: 4px;
}

.collapse-btn:hover {
  background: #3c3c3c;
}

.sidebar-status {
  padding: 8px 12px;
  font-size: 12px;
  flex-shrink: 0;
}

.status-text {
  color: #888;
}

.error-message {
  color: #f44747;
  background: #2d1e1e;
  padding: 6px 8px;
  border-radius: 4px;
}

.file-list {
  list-style: none;
  flex: 1;
  overflow-y: auto;
  overflow-x: hidden;
  margin: 0;
  padding: 0;
  scrollbar-gutter: stable;
}

.file-list::-webkit-scrollbar {
  width: 6px;
}

.file-list::-webkit-scrollbar-track {
  background: transparent;
}

.file-list::-webkit-scrollbar-thumb {
  background: #555;
  border-radius: 3px;
}

.file-list::-webkit-scrollbar-thumb:hover {
  background: #777;
}

.file-list {
  scrollbar-width: thin;
  scrollbar-color: #555 transparent;
}

.file-list li {
  padding: 8px 15px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 13px;
  border-bottom: 1px solid #2a2a2a;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.file-list li:hover {
  background: #2a2d2e;
}

.file-list li.active {
  background: #37373d;
  color: #fff;
  border-left: 2px solid #007acc;
}

.icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 16px;
  height: 16px;
  flex-shrink: 0;
  color: inherit;
}

.file-name {
  overflow: hidden;
  text-overflow: ellipsis;
  flex: 1;
}

.empty {
  color: #666;
  cursor: default;
  font-style: italic;
  padding: 12px 15px;
}

.resize-handle {
  position: absolute;
  top: 0;
  right: 0;
  width: 5px;
  height: 100%;
  cursor: col-resize;
  z-index: 10;
  background: transparent;
  transition: background 0.2s;
}

.resize-handle:hover,
.browser-sidebar.resizing .resize-handle {
  background: #007acc60;
}

:global(body.resizing) {
  cursor: col-resize !important;
  user-select: none !important;
}

/* Collapsed strip */
.collapsed-strip {
  width: 30px;
  background: #2d2d2d;
  border-right: 1px solid #333;
  display: flex;
  align-items: flex-start;
  justify-content: center;
  padding-top: 10px;
  flex-shrink: 0;
}

.expand-btn {
  background: transparent;
  border: none;
  color: #ccc;
  cursor: pointer;
  padding: 4px;
  border-radius: 4px;
}

.expand-btn:hover {
  background: #3c3c3c;
}
</style>