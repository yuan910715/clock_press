<script setup>
import { onBeforeUnmount, ref } from 'vue'

defineProps({
  groupLabel: { type: String, required: true },
  hideLeftLabel: { type: String, required: true },
  showLeftLabel: { type: String, required: true },
  hideRightLabel: { type: String, required: true },
  showRightLabel: { type: String, required: true }
})

const leftHidden = ref(false)
const rightHidden = ref(false)

function setRootClass(name, enabled) {
  if (typeof document !== 'undefined') {
    document.documentElement.classList.toggle(name, enabled)
  }
}

function toggleLeft() {
  leftHidden.value = !leftHidden.value
  setRootClass('api-layout-hide-left', leftHidden.value)
}

function toggleRight() {
  rightHidden.value = !rightHidden.value
  setRootClass('api-layout-hide-right', rightHidden.value)
}

onBeforeUnmount(() => {
  setRootClass('api-layout-hide-left', false)
  setRootClass('api-layout-hide-right', false)
})
</script>

<template>
  <div class="api-layout-controls" role="group" :aria-label="groupLabel">
    <button
      type="button"
      class="api-layout-button api-layout-button--left"
      :aria-pressed="leftHidden"
      @click="toggleLeft"
    >
      {{ leftHidden ? showLeftLabel : hideLeftLabel }}
    </button>
    <button
      type="button"
      class="api-layout-button api-layout-button--right"
      :aria-pressed="rightHidden"
      @click="toggleRight"
    >
      {{ rightHidden ? showRightLabel : hideRightLabel }}
    </button>
  </div>
</template>

<style scoped>
.api-layout-controls {
  position: sticky;
  top: calc(var(--vp-nav-height) + 12px);
  z-index: 20;
  display: none;
  justify-content: flex-end;
  gap: 8px;
  width: fit-content;
  margin: 0 0 18px auto;
  padding: 5px;
  border: 1px solid var(--vp-c-divider);
  border-radius: 10px;
  background: color-mix(in srgb, var(--vp-c-bg) 90%, transparent);
  box-shadow: var(--vp-shadow-1);
  backdrop-filter: blur(10px);
}

.api-layout-button {
  padding: 6px 11px;
  border: 1px solid transparent;
  border-radius: 7px;
  color: var(--vp-c-text-2);
  background: transparent;
  font-size: 13px;
  font-weight: 600;
  line-height: 20px;
  cursor: pointer;
  transition: color 0.2s, border-color 0.2s, background-color 0.2s;
}

.api-layout-button:hover,
.api-layout-button:focus-visible,
.api-layout-button[aria-pressed='true'] {
  border-color: var(--vp-c-divider);
  color: var(--vp-c-brand-1);
  background: var(--vp-c-default-soft);
}

.api-layout-button:focus-visible {
  outline: 2px solid var(--vp-c-brand-1);
  outline-offset: 2px;
}

@media (min-width: 960px) {
  .api-layout-controls {
    display: flex;
  }
}

@media (max-width: 1279px) {
  .api-layout-button--right {
    display: none;
  }
}
</style>
