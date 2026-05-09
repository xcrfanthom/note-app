<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue';
import { readDir, readTextFile, writeTextFile } from '@tauri-apps/plugin-fs';
import { join } from '@tauri-apps/api/path';
import { getCurrentWindow } from '@tauri-apps/api/window';
import NavBar from './components/NavBar.vue';
import SideBar from './components/SideBar.vue';
import Editor from './components/Editor.vue';
import MarkdownEditor from './components/MarkdownEditor.vue';

interface FileItem {
  name: string;
  path: string;
  isDirectory: boolean;
}

interface Tab {
  path: string;
  name: string;
  content: string;
  isDirty: boolean;
  type?: 'text' | 'rich' | 'markdown' | 'tex' | 'draw';
}

const files = ref<FileItem[]>([]);
const currentPath = ref('');
const pathInput = ref('');
const loading = ref(false);
const error = ref('');

const history = ref<string[]>([]);
const historyPos = ref(-1);

const openTabs = ref<Tab[]>([]);
const activeTabIndex = ref(-1);

const canGoBack = computed(() => historyPos.value > 0);
const canGoForward = computed(() => historyPos.value < history.value.length - 1);
const activeFilePath = computed(() =>
  activeTabIndex.value >= 0 ? openTabs.value[activeTabIndex.value]?.path : null
);
const appWindow = getCurrentWindow();

const activeTab = computed(() =>
  activeTabIndex.value >= 0 ? openTabs.value[activeTabIndex.value] : null
);

const activeContent = computed({
  get: () => activeTab.value?.content ?? '',
  set: (val: string) => {
    if (activeTab.value) {
      activeTab.value.content = val;
      activeTab.value.isDirty = true;
    }
  }
});

const activeTabType = computed(() => 
  openTabs.value[activeTabIndex.value]?.type ?? 'text'
);

function pushHistory(path: string) {
  if (historyPos.value < history.value.length - 1) {
    history.value = history.value.slice(0, historyPos.value + 1);
  }
  history.value.push(path);
  historyPos.value = history.value.length - 1;
}

async function toggleFullscreen() {
  const isFullscreen = await appWindow.isFullscreen();
  await appWindow.setFullscreen(!isFullscreen);
}

function handleKeydown(event: KeyboardEvent) {
  if (event.key === 'F11') {
    event.preventDefault();
    toggleFullscreen();
  }
}

async function loadDirectory(targetPath: string, addHistory = true) {
  loading.value = true;
  error.value = '';
  try {
    currentPath.value = targetPath;
    pathInput.value = targetPath;
    const entries = await readDir(targetPath);
    const processed = await Promise.all(
      entries.map(async (entry) => {
        const fullPath = await join(targetPath, entry.name);
        return {
          name: entry.name,
          path: fullPath,
          isDirectory: entry.isDirectory
        } as FileItem;
      })
    );
    files.value = processed.sort(
      (a, b) => Number(b.isDirectory) - Number(a.isDirectory)
    );
    if (addHistory) pushHistory(targetPath);
  } catch (err: any) {
    error.value = err?.message ?? String(err);
  } finally {
    loading.value = false;
  }
}

async function navigateToPath(path: string) {
  if (!path) return;
  await loadDirectory(path, true);
}

function goBack() {
  if (!canGoBack.value) return;
  historyPos.value--;
  loadDirectory(history.value[historyPos.value], false);
}

function goForward() {
  if (!canGoForward.value) return;
  historyPos.value++;
  loadDirectory(history.value[historyPos.value], false);
}

async function goHome() {
  const { homeDir } = await import('@tauri-apps/api/path');
  const home = await homeDir();
  await navigateToPath(home);
}

function refresh() {
  loadDirectory(currentPath.value, false);
}

async function handleFileClick(file: FileItem) {
  if (file.isDirectory) {
    await navigateToPath(file.path);
  } else {
    await openFileInTab(file);
  }
}

async function openFileInTab(file: FileItem) {
  const existingIdx = openTabs.value.findIndex((t) => t.path === file.path);
  if (existingIdx !== -1) {
    activeTabIndex.value = existingIdx;
    return;
  }
  try {
    const content = await readTextFile(file.path);
    const ext = file.name.split('.').pop()?.toLowerCase();
    const newTab: Tab = {
      path: file.path,
      name: file.name,
      content,
      isDirty: false,
      type: ext === 'md' ? 'markdown' : ext === 'tex' ? 'tex' : 'text'
    };
    openTabs.value.push(newTab);
    activeTabIndex.value = openTabs.value.length - 1;
  } catch (err: any) {
    error.value = `Cannot open ${file.name}: ${err?.message ?? err}`;
  }
}

function closeTab(index: number) {
  if (index < 0 || index >= openTabs.value.length) return;
  openTabs.value.splice(index, 1);
  if (activeTabIndex.value >= openTabs.value.length) {
    activeTabIndex.value = openTabs.value.length - 1;
  }
  if (openTabs.value.length === 0) activeTabIndex.value = -1;
}

function selectTab(index: number) {
  if (index >= 0 && index < openTabs.value.length) {
    activeTabIndex.value = index;
  }
}

async function saveFile() {
  if (activeTabIndex.value < 0) return;
  const tab = openTabs.value[activeTabIndex.value];
  try {
    await writeTextFile(tab.path, tab.content);
    tab.isDirty = false;
  } catch (err: any) {
    error.value = `Save failed: ${err?.message ?? err}`;
  }
}

async function createNewFile() {
  const filename = prompt('Enter file name (e.g., note.txt or page.html):');
  if (!filename) return;
  const safeName = filename.includes('.') ? filename : `${filename}.txt`;
  const filePath = await join(currentPath.value, safeName);
  try {
    await writeTextFile(filePath, '');
    await refresh();
    const createdFile: FileItem = {
      name: safeName,
      path: filePath,
      isDirectory: false
    };
    await openFileInTab(createdFile);
  } catch (err: any) {
    error.value = `Failed to create file: ${err?.message ?? err}`;
  }
}

onMounted(async () => {
  const { homeDir } = await import('@tauri-apps/api/path');
  const home = await homeDir();
  await loadDirectory(home, true);
  window.addEventListener('keydown', handleKeydown, true);
});

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown, true);
});
</script>

<template>
  <div class="browser-frame">
    <NavBar
      :path-input="pathInput"
      :can-go-back="canGoBack"
      :can-go-forward="canGoForward"
      @update:path-input="pathInput = $event"
      @navigate="navigateToPath(pathInput)"
      @back="goBack"
      @forward="goForward"
      @home="goHome"
      @refresh="refresh"
    />
    <div class="main-area">
      <SideBar
        :files="files"
        :loading="loading"
        :error="error"
        :active-file-path="activeFilePath"
        @file-click="handleFileClick"
        @create-file="createNewFile"
      />
      <Editor
        v-if="activeTabType === 'text'"
        :tabs="openTabs"
        :active-tab-index="activeTabIndex"
        :active-file-path="activeFilePath"
        :content="activeContent"
        @select-tab="selectTab"
        @close-tab="closeTab"
        @update:content="activeContent = $event"
        @save="saveFile"
      />
      <MarkdownEditor
        v-else-if="activeTabType === 'markdown'"
        :tabs="openTabs"
        :active-tab-index="activeTabIndex"
        :active-file-path="activeFilePath"
        :content="activeContent"
        @select-tab="selectTab"
        @close-tab="closeTab"
        @update:content="activeContent = $event"
        @save="saveFile"
      />
    </div>
  </div>
</template>

<style>
body {
  margin: 0;
  padding: 0;
  overflow: hidden;
}
</style>

<style scoped>
.browser-frame {
  display: flex;
  flex-direction: column;
  height: 100vh;
  width: 100vw;
  background: #1e1e1e;
  color: #d4d4d4;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  overflow: hidden;
}

.main-area {
  display: flex;
  flex: 1;
  overflow: hidden;
}
</style>