<script setup lang="ts">
import { Settings, Sparkles } from 'lucide-vue-next';

defineProps<{
  prompt: string;
  isGenerating: boolean;
  showSettings: boolean;
}>();

const emit = defineEmits<{
  (e: 'update:prompt', value: string): void;
  (e: 'update:showSettings', value: boolean): void;
  (e: 'generate'): void;
}>();
</script>

<template>
  <div class="fixed bottom-0 left-0 right-0 z-30 pb-6 px-4 pointer-events-none">
    <div class="w-full md:max-w-4xl mx-auto pointer-events-auto space-y-3">
      <div 
        class="bg-white dark:bg-zinc-800 rounded-neo-lg shadow-neo-lift dark:shadow-none dark:border dark:border-zinc-700 p-4 md:p-6 transition-all duration-300 ease-out origin-bottom"
        :class="showSettings ? 'opacity-100 scale-100 translate-y-0' : 'max-h-0 opacity-0 scale-95 translate-y-4 overflow-hidden'"
      >
        <slot name="settings"></slot>
      </div>

      <div class="bg-white dark:bg-zinc-800 rounded-neo-lg shadow-neo-lift dark:shadow-none dark:border dark:border-zinc-700 p-4 flex flex-col md:flex-row gap-3 md:gap-4 items-stretch md:items-end transition-colors duration-200">
        <!-- Left side: Presets + References -->
        <div class="flex gap-2 flex-shrink-0 items-end">
          <slot name="presets"></slot>
          <slot name="references"></slot>
        </div>

        <textarea
          :value="prompt"
          @input="emit('update:prompt', ($event.target as HTMLTextAreaElement).value)"
          class="flex-1 bg-zinc-100/80 dark:bg-zinc-700/50 shadow-neo-inset dark:shadow-none rounded-neo px-4 py-3 resize-none outline-none min-h-[80px] md:min-h-[60px] max-h-[200px] focus:bg-white dark:focus:bg-zinc-700 focus:ring-2 focus:ring-teal-100 dark:focus:ring-teal-900 transition-all placeholder-zinc-400 dark:placeholder-zinc-500 text-base text-zinc-900 dark:text-zinc-100"
          placeholder="描述你想要的图像..."
        ></textarea>

        <div class="flex gap-2 flex-shrink-0 justify-end">
          <button 
            @click="emit('update:showSettings', !showSettings)"
            class="p-3 md:p-3 min-w-[44px] min-h-[44px] flex items-center justify-center rounded-neo bg-zinc-100 dark:bg-zinc-700 text-zinc-500 dark:text-zinc-400 hover:bg-zinc-200 dark:hover:bg-zinc-600 hover:scale-105 active:scale-95 transition-all duration-200"
          >
            <Settings class="w-6 h-6 transition-transform duration-500" :class="{ 'rotate-180': showSettings }" />
          </button>
          
          <button 
            @click="emit('generate')"
            :disabled="isGenerating"
            class="px-6 py-3 min-h-[44px] rounded-neo bg-teal-700 dark:bg-teal-600 text-white font-semibold shadow-md hover:bg-teal-800 dark:hover:bg-teal-700 hover:shadow-lg hover:scale-105 active:scale-95 active:shadow-sm transition-all duration-200 disabled:opacity-50 disabled:cursor-not-allowed disabled:hover:scale-100 flex items-center gap-2 group"
          >
            <span v-if="isGenerating" class="flex items-center gap-2">
              <span class="animate-spin h-5 w-5 border-2 border-white border-t-transparent rounded-full"></span>
            </span>
            <span v-else class="flex items-center gap-2">
              生成 <Sparkles class="w-4 h-4 group-hover:rotate-12 transition-transform" />
            </span>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
