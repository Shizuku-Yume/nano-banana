<script setup lang="ts">
interface Props {
  variant?: 'primary' | 'secondary' | 'danger'
  size?: 'sm' | 'md' | 'lg'
  loading?: boolean
  disabled?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  variant: 'primary',
  size: 'md',
  loading: false,
  disabled: false
})

const emit = defineEmits<{
  (e: 'click', event: MouseEvent): void
}>()
</script>

<template>
  <button
    type="button"
    :class="[
      'inline-flex items-center justify-center font-semibold rounded-neo transition-colors',
      {
        'px-3 py-1.5 text-sm': size === 'sm',
        'px-4 py-2 text-sm': size === 'md',
        'px-6 py-3 text-base': size === 'lg',
        'bg-brand text-white hover:bg-brand-dark shadow-md disabled:opacity-50': variant === 'primary',
        'bg-white text-zinc-700 border border-zinc-200 hover:bg-zinc-50 shadow-neo-lift': variant === 'secondary',
        'bg-red-500 text-white hover:bg-red-600 shadow-md': variant === 'danger',
        'cursor-not-allowed opacity-50': disabled || loading
      }
    ]"
    :disabled="disabled || loading"
    @click="(e) => emit('click', e)"
  >
    <div
      v-if="loading"
      class="w-4 h-4 border-2 border-current border-t-transparent rounded-full animate-spin mr-2"
    ></div>
    <slot />
  </button>
</template>
