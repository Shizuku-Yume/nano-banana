<script setup lang="ts">
import type { AspectRatio, Resolution } from '../types'

const props = defineProps<{
  aspectRatios: string[]
  resolution: string
  count: number
}>()

const emit = defineEmits<{
  (e: 'update:aspectRatios', value: string[]): void
  (e: 'update:resolution', value: string): void
  (e: 'update:count', value: number): void
}>()

const availableRatios = [
  '1:1', '2:3', '3:2', '3:4', '4:3',
  '4:5', '5:4', '9:16', '16:9', '21:9'
]

const resolutions = ['1K', '2K', '4K']

const toggleRatio = (ratio: string) => {
  const newRatios = [...props.aspectRatios]
  const index = newRatios.indexOf(ratio)
  if (index === -1) {
    newRatios.push(ratio)
  } else {
    newRatios.splice(index, 1)
  }
  emit('update:aspectRatios', newRatios)
}

const selectResolution = (res: string) => {
  emit('update:resolution', res)
}

const updateCount = (event: Event) => {
  const val = parseInt((event.target as HTMLInputElement).value)
  emit('update:count', val)
}

const getRatioStyle = (ratio: string) => {
  const [w, h] = ratio.split(':').map(Number)
  const maxSize = 24
  const scale = Math.min(maxSize / w, maxSize / h)
  return {
    width: `${w * scale}px`,
    height: `${h * scale}px`
  }
}
</script>

<template>
  <div class="grid grid-cols-1 md:grid-cols-2 gap-6 md:gap-8">
    <div class="flex flex-col gap-4">
      <label class="text-xs uppercase font-bold text-zinc-500 dark:text-zinc-400 tracking-wider">宽高比</label>
      <div class="grid grid-cols-5 gap-2">
        <button
          v-for="ratio in availableRatios"
          :key="ratio"
          @click="toggleRatio(ratio)"
          class="flex flex-col items-center justify-center gap-1.5 p-1 md:p-2 rounded-neo h-14 md:h-16 border transition-all cursor-pointer min-h-[44px]"
          :class="aspectRatios.includes(ratio)
            ? 'bg-brand-light dark:bg-teal-900/30 border-brand dark:border-teal-700 text-brand dark:text-teal-400'
            : 'bg-zinc-50 dark:bg-zinc-800 border-zinc-200 dark:border-zinc-700 text-zinc-500 dark:text-zinc-400 hover:bg-zinc-100 dark:hover:bg-zinc-700'"
        >
          <div class="bg-current rounded-sm opacity-80" :style="getRatioStyle(ratio)"></div>
          <span class="text-[9px] md:text-[10px] font-medium">{{ ratio }}</span>
        </button>
      </div>
    </div>

    <div class="flex flex-col gap-6 md:gap-8">
      <div class="flex flex-col gap-3">
        <label class="text-xs uppercase font-bold text-zinc-500 dark:text-zinc-400 tracking-wider">分辨率</label>
        <div class="flex bg-zinc-100 dark:bg-zinc-800 rounded-neo p-1 gap-1">
          <button
            v-for="res in resolutions"
            :key="res"
            @click="selectResolution(res)"
            class="flex-1 py-2.5 md:py-2 text-sm rounded-lg transition-all font-medium min-h-[40px]"
            :class="resolution === res
              ? 'bg-white dark:bg-zinc-700 shadow-sm text-zinc-800 dark:text-zinc-100'
              : 'text-zinc-500 dark:text-zinc-400 hover:text-zinc-700 dark:hover:text-zinc-200'"
          >
            {{ res }}
          </button>
        </div>
      </div>

      <div class="flex flex-col gap-4">
        <div class="flex justify-between items-center">
          <label class="text-xs uppercase font-bold text-zinc-500 dark:text-zinc-400 tracking-wider">生成数量</label>
          <span class="text-sm font-mono text-brand dark:text-teal-400 font-bold">{{ count }}</span>
        </div>
        <div class="relative h-6 flex items-center">
          <input
            type="range"
            min="1"
            max="8"
            :value="count"
            @input="updateCount"
            class="w-full h-2 bg-zinc-200 dark:bg-zinc-700 rounded-lg appearance-none cursor-pointer accent-brand dark:accent-teal-500"
          />
        </div>
      </div>
    </div>
  </div>
</template>
