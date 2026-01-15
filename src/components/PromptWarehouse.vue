<template>
    <div class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-zinc-900/50 backdrop-blur-sm animate-fade-in" @click.self="$emit('close')">
        <div class="bg-white rounded-neo-lg shadow-neo-lift w-full max-w-6xl h-[85vh] flex flex-col border border-zinc-200 overflow-hidden">
            <!-- Header -->
            <div class="p-4 border-b border-zinc-100 flex justify-between items-center bg-zinc-50">
                <div class="flex items-center gap-3">
                    <h2 class="text-xl font-bold text-zinc-800 flex items-center gap-2">
                        <Library class="w-6 h-6 text-teal-600" /> 提示词仓库
                    </h2>
                    <!-- Mode Toggle -->
                    <div class="flex bg-zinc-200 rounded-full p-0.5">
                        <button
                            @click="currentMode = 'text-to-image'"
                            class="px-3 py-1 text-xs font-medium rounded-full transition-all"
                            :class="currentMode === 'text-to-image' 
                                ? 'bg-white text-teal-700 shadow-sm' 
                                : 'text-zinc-500 hover:text-zinc-700'"
                        >
                            文生图
                        </button>
                        <button
                            @click="currentMode = 'image-to-image'"
                            class="px-3 py-1 text-xs font-medium rounded-full transition-all"
                            :class="currentMode === 'image-to-image' 
                                ? 'bg-white text-teal-700 shadow-sm' 
                                : 'text-zinc-500 hover:text-zinc-700'"
                        >
                            图生图
                        </button>
                    </div>
                </div>
                <button 
                    @click="$emit('close')"
                    class="w-8 h-8 flex items-center justify-center rounded-full hover:bg-zinc-200 transition-colors text-zinc-500"
                >
                    <X class="w-5 h-5" />
                </button>
            </div>

            <!-- Category Filter -->
            <div class="px-4 py-3 border-b border-zinc-100 bg-white overflow-x-auto whitespace-nowrap scrollbar-hide">
                <div class="flex gap-2">
                    <button
                        @click="selectedCategory = 'all'"
                        :class="[
                            'px-4 py-1.5 rounded-full text-sm font-medium transition-all',
                            selectedCategory === 'all'
                                ? 'bg-teal-600 text-white shadow-md'
                                : 'bg-zinc-100 text-zinc-600 hover:bg-zinc-200'
                        ]"
                    >
                        全部
                    </button>
                    <button
                        v-for="cat in categories"
                        :key="cat"
                        @click="selectedCategory = cat"
                        :class="[
                            'px-4 py-1.5 rounded-full text-sm font-medium transition-all',
                            selectedCategory === cat
                                ? 'bg-teal-600 text-white shadow-md'
                                : 'bg-zinc-100 text-zinc-600 hover:bg-zinc-200'
                        ]"
                    >
                        {{ cat }}
                    </button>
                </div>
            </div>

            <!-- Content -->
            <div class="flex-1 overflow-y-auto p-4 bg-zinc-50">
                <div v-if="loading" class="flex flex-col items-center justify-center h-64 gap-4">
                    <div class="w-12 h-12 border-4 border-teal-500 border-t-teal-700 rounded-full animate-spin"></div>
                    <p class="text-zinc-500 font-medium">正在搬运仓库...</p>
                </div>

                <div v-else-if="error" class="flex flex-col items-center justify-center h-64 gap-4">
                    <div class="text-4xl">😵</div>
                    <p class="text-red-600 font-medium">{{ error }}</p>
                    <button @click="fetchPrompts" class="text-teal-600 hover:text-teal-700 underline">重试</button>
                </div>

                <div v-else-if="filteredPrompts.length === 0" class="flex flex-col items-center justify-center h-64 text-zinc-500">
                    <div class="text-4xl mb-2">📭</div>
                    <p>该分类下暂无提示词</p>
                </div>

                <div v-else class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4">
                    <div 
                        v-for="item in filteredPrompts" 
                        :key="item.title"
                        class="bg-white rounded-neo border border-zinc-200 overflow-hidden hover:shadow-neo-float transition-all duration-300 group flex flex-col"
                    >
                        <!-- Image Preview -->
                        <div class="aspect-square bg-zinc-100 relative overflow-hidden">
                            <img 
                                :src="item.preview" 
                                :alt="item.title"
                                loading="lazy"
                                class="w-full h-full object-cover group-hover:scale-105 transition-transform duration-700"
                            />
                            <!-- Hover Overlay -->
                            <div class="absolute inset-0 bg-black/20 opacity-0 group-hover:opacity-100 transition-opacity flex items-center justify-center gap-2 backdrop-blur-[2px]">
                                <button 
                                    @click="usePrompt(item)"
                                    class="bg-white text-teal-700 px-5 py-2 rounded-full font-bold text-sm hover:bg-teal-600 hover:text-white transition-all transform translate-y-2 group-hover:translate-y-0 duration-300 shadow-lg"
                                >
                                    立即使用
                                </button>
                            </div>
                        </div>

                        <!-- Info -->
                        <div class="p-3 flex flex-col gap-2 flex-1">
                            <div class="flex justify-between items-start gap-2">
                                <h3 class="font-bold text-zinc-800 line-clamp-1" :title="item.title">{{ item.title }}</h3>
                                <button 
                                    @click="savePrompt(item)"
                                    class="text-zinc-400 hover:text-teal-500 transition-colors"
                                    title="收藏到预设库"
                                >
                                    <Star class="w-4 h-4" />
                                </button>
                            </div>
                            <p class="text-xs text-zinc-500 line-clamp-2" :title="item.prompt">
                                {{ item.prompt }}
                            </p>
                            
                            <!-- Tags -->
                            <div class="mt-auto pt-2 flex gap-1 flex-wrap">
                                <span class="text-[10px] bg-zinc-100 text-zinc-500 px-2 py-1 rounded-full border border-zinc-200">
                                    {{ item.category || '通用' }}
                                </span>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, onMounted, computed } from 'vue'
import { Library, X, Star } from 'lucide-vue-next'
import type { StyleTemplate, GenerationMode } from '../types'

interface WarehouseItem {
    title: string
    prompt: string
    preview: string
    category?: string
    mode?: 'generate' | 'edit'
}

const props = defineProps<{
    mode?: GenerationMode
}>()

const emit = defineEmits<{
    'close': []
    'use-prompt': [prompt: string]
    'save-prompt': [template: StyleTemplate]
}>()

const currentMode = ref<GenerationMode>(props.mode || 'text-to-image')
const prompts = ref<WarehouseItem[]>([])
const loading = ref(true)
const error = ref<string | null>(null)
const selectedCategory = ref('all')

const categories = computed(() => {
    const cats = new Set<string>()
    const targetMode = currentMode.value === 'text-to-image' ? 'generate' : 'edit'
    const modePrompts = prompts.value.filter(p => !p.mode || p.mode === targetMode)
    
    modePrompts.forEach(p => {
        if (p.category) cats.add(p.category)
    })
    return Array.from(cats).sort()
})

const filteredPrompts = computed(() => {
    const targetMode = currentMode.value === 'text-to-image' ? 'generate' : 'edit'
    
    return prompts.value.filter(item => {
        const modeMatch = !item.mode || item.mode === targetMode
        if (!modeMatch) return false

        if (selectedCategory.value !== 'all' && item.category !== selectedCategory.value) {
            return false
        }

        return true
    })
})

const fetchPrompts = async () => {
    loading.value = true
    error.value = null
    try {
        const response = await fetch('https://raw.githubusercontent.com/glidea/banana-prompt-quicker/main/prompts.json')
        if (!response.ok) throw new Error('Failed to fetch prompts')
        const data = await response.json()
        prompts.value = data
    } catch (e) {
        error.value = e instanceof Error ? e.message : '加载失败'
    } finally {
        loading.value = false
    }
}

const usePrompt = (item: WarehouseItem) => {
    emit('use-prompt', item.prompt)
    emit('close')
}

const savePrompt = (item: WarehouseItem) => {
    const template: StyleTemplate = {
        id: `warehouse-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
        title: item.title,
        prompt: item.prompt,
        description: item.category || '来自提示词仓库',
        image: '',
        mode: currentMode.value
    }
    emit('save-prompt', template)
}

onMounted(() => {
    fetchPrompts()
})
</script>

<style scoped>
.scrollbar-hide::-webkit-scrollbar {
    display: none;
}
.scrollbar-hide {
    -ms-overflow-style: none;
    scrollbar-width: none;
}
</style>
