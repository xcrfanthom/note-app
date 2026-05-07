<script setup lang="ts">
defineProps<{
  pathInput: string;
  canGoBack: boolean;
  canGoForward: boolean;
}>();

defineEmits<{
  (e: 'update:pathInput', value: string): void;
  (e: 'navigate'): void;
  (e: 'back'): void;
  (e: 'forward'): void;
  (e: 'home'): void;
  (e: 'refresh'): void;
}>();
</script>

<template>
  <header class="nav-bar">
    <div class="nav-buttons">
      <button @click="$emit('back')" :disabled="!canGoBack" title="Back">←</button>
      <button @click="$emit('forward')" :disabled="!canGoForward" title="Forward">→</button>
      <button @click="$emit('home')" title="Home">⌂</button>
      <button @click="$emit('refresh')" title="Refresh">↻</button>
    </div>
    <div class="address-bar">
      <input
        :value="pathInput"
        @input="$emit('update:pathInput', ($event.target as HTMLInputElement).value)"
        @keyup.enter="$emit('navigate')"
        type="text"
        placeholder="Enter path..."
      />
      <button @click="$emit('navigate')" class="go-btn">Go</button>
    </div>
  </header>
</template>

<style scoped>
.nav-bar {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 12px;
  background: #2d2d2d;
  border-bottom: 1px solid #3c3c3c;
}

.nav-buttons {
  display: flex;
  gap: 4px;
}

.nav-buttons button {
  background: #3c3c3c;
  border: none;
  color: #ccc;
  font-size: 16px;
  padding: 4px 10px;
  border-radius: 4px;
  cursor: pointer;
  transition: background 0.2s;
}

.nav-buttons button:hover:not(:disabled) {
  background: #505050;
}

.nav-buttons button:disabled {
  opacity: 0.5;
  cursor: default;
}

.address-bar {
  flex: 1;
  display: flex;
  gap: 4px;
}

.address-bar input {
  flex: 1;
  background: #1e1e1e;
  border: 1px solid #3c3c3c;
  color: #d4d4d4;
  padding: 6px 12px;
  border-radius: 4px;
  font-size: 13px;
}

.go-btn {
  background: #007acc;
  color: white;
  border: none;
  padding: 6px 16px;
  border-radius: 4px;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s;
}

.go-btn:hover {
  background: #108ad1;
}
</style>