<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'
import type { GeneratedImage } from '../types'
import ImageCard from './ImageCard.vue'

const props = defineProps<{
  images: GeneratedImage[]
  loading: boolean
}>()

const emit = defineEmits<{
  (e: 'openLightbox', images: GeneratedImage[], index: number): void
  (e: 'toggleFavorite', id: number): void
  (e: 'deleteImage', id: number): void
  (e: 'iterate', image: GeneratedImage): void
  (e: 'loadMore'): void
}>()

const sentinel = ref<HTMLElement | null>(null)
let observer: IntersectionObserver | null = null

const setupObserver = () => {
  if (observer) observer.disconnect()
  
  observer = new IntersectionObserver((entries) => {
    if (entries[0].isIntersecting && !props.loading) {
      emit('loadMore')
    }
  }, { rootMargin: '200px' })
  
  if (sentinel.value) observer.observe(sentinel.value)
}

onMounted(() => {
  setupObserver()
})

onUnmounted(() => {
  if (observer) observer.disconnect()
})
</script>

<template>
  <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 xl:grid-cols-5 gap-4">
    <ImageCard 
      v-for="(img, index) in images" 
      :key="img.id || index"
      :image="img"
      status="success"
      :aspect-ratio="img.aspectRatio"
      class="animate-fade-in"
      :style="{ animationDelay: `${(index % 10) * 50}ms` }"
      @click="emit('openLightbox', images, index)"
      @favorite="emit('toggleFavorite', $event)"
      @delete="emit('deleteImage', $event)"
      @iterate="emit('iterate', $event)"
    />
    
    <template v-if="loading">
      <div 
        v-for="i in 8" 
        :key="`skeleton-${i}`" 
        class="bg-zinc-200 dark:bg-zinc-800 border border-zinc-300 dark:border-zinc-700 rounded-neo-lg animate-pulse aspect-square shadow-neo-inset dark:shadow-none"
      ></div>
    </template>

    <div ref="sentinel" class="col-span-full h-4 w-full opacity-0 pointer-events-none"></div>
  </div>
</template>
