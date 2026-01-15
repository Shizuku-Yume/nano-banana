<script setup lang="ts">
import { Plus, Library } from 'lucide-vue-next'
import { StylePreset } from '../types'

defineProps<{
  presets: StylePreset[]
  selectedId: string | null
}>()

const emit = defineEmits<{
  select: [id: string | null]
  create: []
  openWarehouse: []
}>()
</script>

<template>
  <div class="flex gap-2 overflow-x-auto pb-1 scrollbar-hide">
    <!-- None Option -->
    <button
      @click="emit('select', null)"
      class="flex items-center gap-2 px-4 py-2 rounded-full text-sm whitespace-nowrap border transition-all cursor-pointer"
      :class="selectedId === null 
        ? 'bg-brand-light border-brand text-brand' 
        : 'bg-white border-zinc-200 text-zinc-600 hover:border-zinc-300'"
    >
      None
    </button>

    <!-- Presets -->
    <button
      v-for="preset in presets"
      :key="preset.id"
      @click="emit('select', String(preset.id))"
      class="flex items-center gap-2 px-4 py-2 rounded-full text-sm whitespace-nowrap border transition-all cursor-pointer"
      :class="String(preset.id) === selectedId 
        ? 'bg-brand-light border-brand text-brand' 
        : 'bg-white border-zinc-200 text-zinc-600 hover:border-zinc-300'"
    >
      {{ preset.name }}
    </button>

    <!-- Create New -->
    <button
      @click="emit('create')"
      class="flex items-center gap-2 px-4 py-2 rounded-full text-sm whitespace-nowrap border transition-all cursor-pointer bg-white border-zinc-200 text-zinc-600 hover:border-zinc-300"
      title="Create new preset"
    >
      <Plus class="w-4 h-4" />
    </button>

    <!-- Prompt Warehouse -->
    <button
      @click="emit('openWarehouse')"
      class="flex items-center gap-2 px-4 py-2 rounded-full text-sm whitespace-nowrap border transition-all cursor-pointer bg-teal-50 border-teal-200 text-teal-700 hover:bg-teal-100"
      title="Open prompt warehouse"
    >
      <Library class="w-4 h-4" />
      <span class="hidden sm:inline">仓库</span>
    </button>
  </div>
</template>

<style scoped>
.scrollbar-hide::-webkit-scrollbar {
  display: none;
}
.scrollbar-hide {
  -ms-overflow-style: none;
  scrollbar-width: none;
}
</style>
