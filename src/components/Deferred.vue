<!-- components/Deferred.vue -->
<template>
  <div ref="rootEl" class="deferred">
     <!-- @slot Slot used for rendering the full document shown in a cluster
        -->
    <slot v-if="renderNow" />
     <!-- @slot Slot used for lightweight preview rendering of document in a cluster.
        This typically is a fixed width element and uses simple gray lines to suggest 
        some text content. Using these lightweight elements as the user is moving a 
        cluster slider can rapidly improve performance.
        -->
    <slot v-else name="placeholder" />
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'

const rootEl = ref<HTMLElement | null>(null)
const isVisible = ref(false)
const pastDelay = ref(false)
const isIdle = ref(false)

const renderNow = computed(() => isVisible.value && pastDelay.value && isIdle.value)

let io: IntersectionObserver | null = null
let delayTimer: number | null = null
// let idleHandle: number | null = null
let idleTimeoutFallback: number | null = null

onMounted(() => {
  // 1) Visibility gate
  if (rootEl.value) {
    io = new IntersectionObserver((entries) => {
      for (const e of entries) {
        if (e.isIntersecting) {
          isVisible.value = true
          if (io) { io.disconnect(); io = null }
        }
      }
    }, { rootMargin: '300px' })
    io.observe(rootEl.value)
  }

  // 2) Small delay
  delayTimer = window.setTimeout(() => { pastDelay.value = true }, 100)

  // 3) Idle gate
  const win = window as any
  if (typeof win.requestIdleCallback === 'function') {
    // idleHandle = win.requestIdleCallback(
    win.requestIdleCallback(
      () => { isIdle.value = true },
      { timeout: 500 }
    )
  } else {
    // Fallback when rIC isn't available
    idleTimeoutFallback = window.setTimeout(() => { isIdle.value = true }, 200)
  }
})

onBeforeUnmount(() => {
  if (io) io.disconnect()
  if (delayTimer) window.clearTimeout(delayTimer)
  if (idleTimeoutFallback) window.clearTimeout(idleTimeoutFallback)
  // (No need to cancel requestIdleCallback explicitly here)
})
</script>

<style scoped>
.deferred {
  contain: content;
  content-visibility: auto; /* lets the browser skip paint until needed */
  /* Sizing is provided by your placeholder slot to avoid layout shift */
}
</style>