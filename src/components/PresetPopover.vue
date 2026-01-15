<script setup lang="ts">
import { ref, computed } from 'vue'
import { BookMarked, Plus, Library, X, ChevronDown, Pencil, Trash2 } from 'lucide-vue-next'
import type { StylePreset } from '../types'

const props = defineProps<{
  presets: StylePreset[]
  selectedId: string | null
}>()

const emit = defineEmits<{
  select: [id: string | null]
  apply: [preset: StylePreset]
  edit: [preset: StylePreset]
  delete: [id: number]
  create: []
  openWarehouse: []
}>()

const isOpen = ref(false)

const selectedPreset = computed(() => {
  if (!props.selectedId) return null
  return props.presets.find(p => String(p.id) === props.selectedId)
})

const togglePopover = () => {
  isOpen.value = !isOpen.value
}

const handleSelectPreset = (preset: StylePreset) => {
  emit('select', String(preset.id))
  emit('apply', preset)
  isOpen.value = false
}

const handleClearSelection = () => {
  emit('select', null)
  isOpen.value = false
}

const handleEdit = (e: Event, preset: StylePreset) => {
  e.stopPropagation()
  emit('edit', preset)
  isOpen.value = false
}

const handleDelete = (e: Event, id: number) => {
  e.stopPropagation()
  if (confirm('Delete this preset?')) {
    emit('delete', id)
  }
}

const handleCreate = () => {
  emit('create')
  isOpen.value = false
}

const handleOpenWarehouse = () => {
  emit('openWarehouse')
  isOpen.value = false
}
</script>

<template>
  <div class="relative">
    <!-- Trigger Button -->
    <button
      @click="togglePopover"
      class="flex items-center gap-2 px-3 py-2 rounded-neo border transition-all min-h-[44px]"
      :class="[
        selectedPreset 
          ? 'bg-teal-50 border-teal-200 text-teal-700' 
          : 'bg-zinc-100 border-zinc-200 text-zinc-600 hover:bg-zinc-200'
      ]"
    >
      <BookMarked class="w-4 h-4" />
      <span class="text-sm font-medium max-w-[100px] truncate">
        {{ selectedPreset?.name || 'Presets' }}
      </span>
      <ChevronDown 
        class="w-4 h-4 transition-transform" 
        :class="{ 'rotate-180': isOpen }" 
      />
    </button>

    <!-- Popover -->
    <Transition
      enter-active-class="transition ease-out duration-200"
      enter-from-class="opacity-0 translate-y-2"
      enter-to-class="opacity-100 translate-y-0"
      leave-active-class="transition ease-in duration-150"
      leave-from-class="opacity-100 translate-y-0"
      leave-to-class="opacity-0 translate-y-2"
    >
      <div
        v-if="isOpen"
        class="absolute bottom-full left-0 mb-2 w-80 bg-white rounded-neo-lg shadow-neo-lift border border-zinc-200 overflow-hidden z-50"
      >
        <!-- Header -->
        <div class="p-3 border-b border-zinc-100 flex items-center justify-between bg-zinc-50">
          <h3 class="text-sm font-bold text-zinc-800">Prompt Presets</h3>
          <button
            @click="isOpen = false"
            class="p-1 rounded hover:bg-zinc-200 text-zinc-400"
          >
            <X class="w-4 h-4" />
          </button>
        </div>

        <!-- Preset List -->
        <div class="max-h-64 overflow-y-auto p-2">
          <!-- None Option -->
          <button
            @click="handleClearSelection"
            class="w-full text-left px-3 py-2 rounded-lg text-sm transition-colors flex items-center gap-3"
            :class="[
              !selectedId 
                ? 'bg-teal-50 text-teal-700' 
                : 'hover:bg-zinc-100 text-zinc-600'
            ]"
          >
            <div class="w-8 h-8 rounded bg-zinc-200 flex items-center justify-center text-zinc-400">
              <X class="w-4 h-4" />
            </div>
            <span class="font-medium">None</span>
          </button>

          <!-- Presets -->
          <div
            v-for="preset in presets"
            :key="preset.id"
            class="mt-1 rounded-lg transition-colors group"
            :class="[
              String(preset.id) === selectedId 
                ? 'bg-teal-50' 
                : 'hover:bg-zinc-100'
            ]"
          >
            <div class="flex items-center gap-3 px-3 py-2">
              <!-- Click to apply -->
              <button
                @click="handleSelectPreset(preset)"
                class="flex items-center gap-3 flex-1 min-w-0 text-left"
                :class="String(preset.id) === selectedId ? 'text-teal-700' : 'text-zinc-600'"
              >
                <!-- Thumbnail or placeholder -->
                <div class="w-8 h-8 rounded overflow-hidden flex-shrink-0">
                  <img 
                    v-if="preset.referenceImages?.[0]"
                    :src="preset.referenceImages[0]"
                    alt=""
                    class="w-full h-full object-cover"
                  />
                  <div v-else class="w-full h-full bg-gradient-to-br from-teal-100 to-teal-200 flex items-center justify-center">
                    <BookMarked class="w-3 h-3 text-teal-500" />
                  </div>
                </div>
                <div class="flex-1 min-w-0">
                  <p class="font-medium text-sm truncate">{{ preset.name }}</p>
                  <p v-if="preset.description" class="text-xs text-zinc-400 truncate">{{ preset.description }}</p>
                </div>
              </button>

              <!-- Action Buttons -->
              <div class="flex gap-1 opacity-0 group-hover:opacity-100 transition-opacity">
                <button
                  @click="handleEdit($event, preset)"
                  class="p-1.5 rounded hover:bg-zinc-200 text-zinc-400 hover:text-zinc-600"
                  title="Edit"
                >
                  <Pencil class="w-3.5 h-3.5" />
                </button>
                <button
                  @click="handleDelete($event, preset.id!)"
                  class="p-1.5 rounded hover:bg-red-100 text-zinc-400 hover:text-red-600"
                  title="Delete"
                >
                  <Trash2 class="w-3.5 h-3.5" />
                </button>
              </div>
            </div>
          </div>

          <!-- Empty State -->
          <div v-if="presets.length === 0" class="py-6 text-center text-zinc-400">
            <BookMarked class="w-8 h-8 mx-auto mb-2 opacity-50" />
            <p class="text-sm">No presets yet</p>
          </div>
        </div>

        <!-- Actions -->
        <div class="p-2 border-t border-zinc-100 flex gap-2">
          <button
            @click="handleCreate"
            class="flex-1 flex items-center justify-center gap-2 px-3 py-2 rounded-lg text-sm font-medium bg-zinc-100 text-zinc-600 hover:bg-zinc-200 transition-colors"
          >
            <Plus class="w-4 h-4" />
            Create New
          </button>
          <button
            @click="handleOpenWarehouse"
            class="flex-1 flex items-center justify-center gap-2 px-3 py-2 rounded-lg text-sm font-medium bg-teal-50 text-teal-700 hover:bg-teal-100 transition-colors"
          >
            <Library class="w-4 h-4" />
            Warehouse
          </button>
        </div>
      </div>
    </Transition>

    <!-- Backdrop -->
    <div
      v-if="isOpen"
      class="fixed inset-0 z-40"
      @click="isOpen = false"
    ></div>
  </div>
</template>
