<script setup lang="ts">
import { Sparkles, Image, Heart } from 'lucide-vue-next';
import type { TabType } from '../types';
import ThemeToggle from './ThemeToggle.vue';

defineProps<{
  activeTab: TabType;
  isConnected: boolean;
}>();

const emit = defineEmits<{
  (e: 'update:activeTab', tab: TabType): void;
  (e: 'openApiConfig'): void;
}>();

const tabs = [
  { id: 'create', label: '创作', icon: Sparkles },
  { id: 'gallery', label: '图库', icon: Image },
  { id: 'favorites', label: '收藏', icon: Heart },
] as const;
</script>

<template>
  <header class="fixed top-0 left-0 right-0 z-40 h-16 bg-white/95 dark:bg-zinc-900/95 backdrop-blur-sm shadow-neo-lift dark:shadow-none dark:border-b dark:border-zinc-800 flex items-center justify-between px-4 md:px-6 transition-colors duration-200">
    <div class="flex items-center gap-2 md:gap-3 cursor-pointer group">
      <span class="text-2xl md:text-3xl transition-transform duration-300 group-hover:rotate-12">🍌</span>
      <div class="flex flex-col">
        <span class="text-lg md:text-xl font-bold text-zinc-900 dark:text-zinc-100 leading-none">Nano Banana</span>
        <span class="text-[9px] md:text-[10px] text-zinc-400 dark:text-zinc-500 font-medium tracking-wide uppercase hidden sm:inline-block">AI 图像生成套件</span>
      </div>
    </div>

    <!-- Mobile Nav (Icon only) -->
    <nav class="flex md:hidden bg-zinc-100 dark:bg-zinc-800 rounded-neo p-1 gap-1">
      <button
        v-for="tab in tabs"
        :key="tab.id"
        @click="emit('update:activeTab', tab.id as TabType)"
        class="p-2 rounded-lg text-sm font-medium transition-all duration-200 min-h-[40px] min-w-[40px] flex items-center justify-center relative"
        :class="[
          activeTab === tab.id 
            ? 'bg-white dark:bg-zinc-700 shadow-sm text-brand dark:text-teal-400 scale-100' 
            : 'text-zinc-500 dark:text-zinc-400 hover:text-zinc-700 dark:hover:text-zinc-200 hover:bg-zinc-200/50 dark:hover:bg-zinc-700/50 scale-95'
        ]"
      >
        <component :is="tab.icon" class="w-5 h-5 z-10" />
        <span v-if="activeTab === tab.id" class="absolute bottom-1 w-1 h-1 bg-brand dark:bg-teal-400 rounded-full"></span>
      </button>
    </nav>

    <!-- Desktop Nav (Labels + Icons) -->
    <nav class="absolute left-1/2 -translate-x-1/2 hidden md:flex bg-zinc-100 dark:bg-zinc-800 rounded-neo p-1 gap-1">
      <button
        v-for="tab in tabs"
        :key="tab.id"
        @click="emit('update:activeTab', tab.id as TabType)"
        class="flex items-center gap-2 px-4 py-2 rounded-lg text-sm font-medium transition-all duration-200 relative overflow-hidden"
        :class="[
          activeTab === tab.id 
            ? 'bg-white dark:bg-zinc-700 shadow-sm text-brand dark:text-teal-400 scale-100' 
            : 'text-zinc-500 dark:text-zinc-400 hover:text-zinc-700 dark:hover:text-zinc-200 hover:bg-zinc-200/50 dark:hover:bg-zinc-700/50 scale-95'
        ]"
      >
        <component :is="tab.icon" class="w-4 h-4 z-10" />
        <span class="z-10">{{ tab.label }}</span>
        <span v-if="activeTab === tab.id" class="absolute bottom-0 left-0 h-0.5 bg-brand dark:bg-teal-400 w-full animate-scale-in"></span>
      </button>
    </nav>

    <div class="flex items-center gap-2 md:gap-3">
      <ThemeToggle class="hidden sm:flex" />
      
      <button
        @click="emit('openApiConfig')"
        class="flex items-center gap-2 px-3 py-2 md:px-4 rounded-full text-xs font-medium border transition-all hover:shadow-sm active:scale-95 hover:scale-105 min-h-[36px]"
        :class="[
          isConnected 
            ? 'bg-teal-50 dark:bg-teal-900/30 border-teal-200 dark:border-teal-800 text-teal-700 dark:text-teal-400 hover:bg-teal-100 dark:hover:bg-teal-900/50' 
            : 'bg-red-50 dark:bg-red-900/30 border-red-200 dark:border-red-800 text-red-600 dark:text-red-400 hover:bg-red-100 dark:hover:bg-red-900/50'
        ]"
      >
        <span 
          class="w-2 h-2 rounded-full flex-shrink-0" 
          :class="isConnected ? 'bg-teal-500 animate-pulse' : 'bg-red-500'"
        ></span>
        <span class="hidden sm:inline">{{ isConnected ? '已连接' : '未连接' }}</span>
        <span class="sm:hidden">{{ isConnected ? 'API' : '无 API' }}</span>
      </button>
    </div>
  </header>
</template>
