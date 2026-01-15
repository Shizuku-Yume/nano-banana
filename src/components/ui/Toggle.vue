<script setup lang="ts">
defineProps<{
  modelValue: boolean
  label?: string
}>()

defineEmits<{
  (e: 'update:modelValue', value: boolean): void
}>()
</script>

<template>
  <div class="flex items-center gap-3 cursor-pointer" @click="$emit('update:modelValue', !modelValue)">
    <button
      type="button"
      role="switch"
      :aria-checked="modelValue"
      class="relative w-11 h-6 rounded-full transition-colors"
      :class="modelValue ? 'bg-brand' : 'bg-zinc-200'"
    >
      <span
        class="absolute top-0.5 left-0.5 w-5 h-5 bg-white rounded-full shadow transition-transform"
        :class="modelValue ? 'translate-x-5' : ''"
      ></span>
    </button>
    <span v-if="label" class="text-sm text-zinc-700">{{ label }}</span>
  </div>
</template>
