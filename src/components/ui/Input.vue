<script setup lang="ts">
withDefaults(defineProps<{
  modelValue: string
  label?: string
  placeholder?: string
  type?: string
  disabled?: boolean
}>(), {
  type: 'text',
  disabled: false
})

const emit = defineEmits<{
  (e: 'update:modelValue', value: string): void
}>()
</script>

<template>
  <div>
    <label v-if="label" class="text-sm font-medium text-zinc-700 block mb-1">
      {{ label }}
    </label>
    <input
      :value="modelValue"
      :type="type"
      :placeholder="placeholder"
      :disabled="disabled"
      class="w-full bg-zinc-100/80 shadow-neo-inset rounded-neo px-4 py-2.5 outline-none transition-all focus:bg-white focus:ring-2 focus:ring-teal-100 text-zinc-900 placeholder:text-zinc-400 disabled:opacity-50 disabled:cursor-not-allowed"
      @input="emit('update:modelValue', ($event.target as HTMLInputElement).value)"
    >
  </div>
</template>
