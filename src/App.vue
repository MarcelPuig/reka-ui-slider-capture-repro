<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'
import { SliderRange, SliderRoot, SliderThumb, SliderTrack } from 'reka-ui'

const value = ref([20, 80])
const commits = ref<string[]>([])
const dropCapture = ref(true)

// Chrome sometimes fires `lostpointercapture` before `pointerup`. Releasing capture from a
// capture-phase `pointerup` listener reproduces that order on every drag.
function dropCaptureBeforeRelease(event: PointerEvent) {
  const target = event.target as HTMLElement
  if (dropCapture.value && target.hasPointerCapture?.(event.pointerId)) {
    target.releasePointerCapture(event.pointerId)
  }
}
onMounted(() => window.addEventListener('pointerup', dropCaptureBeforeRelease, true))
onUnmounted(() => window.removeEventListener('pointerup', dropCaptureBeforeRelease, true))
</script>

<template>
  <main style="font-family: sans-serif; padding: 40px; max-width: 420px">
    <label><input v-model="dropCapture" type="checkbox"> Lose capture before pointerup</label>
    <p>Value: {{ value }}</p>
    <SliderRoot
      v-model="value"
      :max="100"
      :step="1"
      style="position: relative; display: flex; align-items: center; height: 20px; margin: 24px 0"
      @value-commit="commits.unshift(`valueCommit ${JSON.stringify($event)}`)"
    >
      <SliderTrack style="position: relative; flex-grow: 1; height: 4px; background: #ddd">
        <SliderRange style="position: absolute; height: 100%; background: #333" />
      </SliderTrack>
      <SliderThumb
        v-for="index in 2"
        :key="index"
        aria-label="Price"
        style="display: block; width: 20px; height: 20px; border-radius: 50%; background: white; border: 2px solid #333"
      />
    </SliderRoot>
    <p>Drag a thumb and release. With the box ticked, the value changes but no valueCommit is logged,
      and the next drag starts offset from the pointer.</p>
    <ol>
      <li v-for="(commit, index) in commits" :key="index">{{ commit }}</li>
    </ol>
  </main>
</template>
