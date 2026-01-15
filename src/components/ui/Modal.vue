<script setup lang="ts">
import { onMounted, onUnmounted, computed } from 'vue'
import { X } from 'lucide-vue-next'

const props = withDefaults(defineProps<{
  open?: boolean
  show?: boolean
  title?: string
  width?: string
  closable?: boolean
}>(), {
  closable: true
})

const isOpen = computed(() => props.open || props.show)

const emit = defineEmits<{
  (e: 'close'): void
}>()

const handleClose = () => {
  if (props.closable) {
    emit('close')
  }
}

const handleKeydown = (e: KeyboardEvent) => {
  if (e.key === 'Escape' && isOpen.value) {
    handleClose()
  }
}

onMounted(() => {
  document.addEventListener('keydown', handleKeydown)
})

onUnmounted(() => {
  document.removeEventListener('keydown', handleKeydown)
})
</script>

<template>
  <Teleport to="body">
    <div
      v-if="isOpen"
      class="fixed inset-0 z-50 flex items-center justify-center bg-zinc-900/50 backdrop-blur-sm animate-fade-in"
      @click="handleClose"
    >
      <div
        class="relative w-full mx-4 bg-white rounded-neo-lg shadow-2xl p-6"
        :class="width || 'max-w-lg'"
        @click.stop
      >
        <div class="flex items-center justify-between mb-4">
          <h2 v-if="title" class="text-xl font-bold text-zinc-900">
            {{ title }}
          </h2>
          <button
            v-if="closable"
            @click="handleClose"
            class="p-2 text-zinc-400 hover:text-zinc-600 rounded-full hover:bg-zinc-100 transition-colors"
          >
            <X class="w-5 h-5" />
          </button>
        </div>
        
        <div class="text-zinc-900">
          <slot />
        </div>
      </div>
    </div>
  </Teleport>
</template>
