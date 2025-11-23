<template>
    <div class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/50 backdrop-blur-sm animate-fade-in" @click.self="$emit('close')">
        <div class="bg-white rounded-xl shadow-2xl w-full max-w-6xl h-[85vh] flex flex-col border-4 border-black overflow-hidden">
            <!-- Header -->
            <div class="p-4 border-b-2 border-gray-100 flex justify-between items-center bg-gray-50">
                <div class="flex items-center gap-2">
                    <h2 class="text-xl font-black flex items-center gap-2">
                        🏪 提示词仓库
                        <span class="text-sm font-normal text-gray-500 bg-gray-200 px-2 py-0.5 rounded-full">
                            {{ mode === 'text-to-image' ? '文生图' : '图文生图' }}
                        </span>
                    </h2>
                </div>
                <button 
                    @click="$emit('close')"
                    class="w-8 h-8 flex items-center justify-center rounded-full hover:bg-gray-200 transition-colors font-bold text-gray-500"
                >
                    ✕
                </button>
            </div>

            <!-- Category Filter -->
            <div class="px-4 py-3 border-b border-gray-100 bg-white overflow-x-auto whitespace-nowrap scrollbar-hide">
                <div class="flex gap-2">
                    <button
                        @click="selectedCategory = 'all'"
                        :class="[
                            'px-4 py-1.5 rounded-full text-sm font-bold transition-all border-2',
                            selectedCategory === 'all'
                                ? 'bg-black text-white border-black'
                                : 'bg-gray-100 text-gray-600 border-transparent hover:bg-gray-200'
                        ]"
                    >
                        全部
                    </button>
                    <button
                        v-for="cat in categories"
                        :key="cat"
                        @click="selectedCategory = cat"
                        :class="[
                            'px-4 py-1.5 rounded-full text-sm font-bold transition-all border-2',
                            selectedCategory === cat
                                ? 'bg-yellow-400 text-black border-black'
                                : 'bg-gray-100 text-gray-600 border-transparent hover:bg-gray-200'
                        ]"
                    >
                        {{ cat }}
                    </button>
                </div>
            </div>

            <!-- Content -->
            <div class="flex-1 overflow-y-auto p-4 bg-gray-50">
                <div v-if="loading" class="flex flex-col items-center justify-center h-64 gap-4">
                    <div class="w-12 h-12 border-4 border-yellow-400 border-t-black rounded-full animate-spin"></div>
                    <p class="text-gray-500 font-bold">正在搬运仓库...</p>
                </div>

                <div v-else-if="error" class="flex flex-col items-center justify-center h-64 gap-4">
                    <div class="text-4xl">😵</div>
                    <p class="text-red-500 font-bold">{{ error }}</p>
                    <button @click="fetchPrompts" class="text-blue-500 underline">重试</button>
                </div>

                <div v-else-if="filteredPrompts.length === 0" class="flex flex-col items-center justify-center h-64 text-gray-500">
                    <div class="text-4xl mb-2">📭</div>
                    <p>该分类下暂无提示词</p>
                </div>

                <div v-else class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4">
                    <div 
                        v-for="item in filteredPrompts" 
                        :key="item.title"
                        class="bg-white rounded-lg border border-gray-200 overflow-hidden hover:shadow-lg transition-all group flex flex-col"
                    >
                        <!-- Image Preview -->
                        <div class="aspect-square bg-gray-100 relative overflow-hidden">
                            <img 
                                :src="item.preview" 
                                :alt="item.title"
                                loading="lazy"
                                class="w-full h-full object-cover group-hover:scale-105 transition-transform duration-500"
                            />
                            <!-- Hover Overlay -->
                            <div class="absolute inset-0 bg-black/40 opacity-0 group-hover:opacity-100 transition-opacity flex items-center justify-center gap-2">
                                <button 
                                    @click="usePrompt(item)"
                                    class="bg-white text-black px-4 py-2 rounded-full font-bold text-sm hover:bg-yellow-400 transition-colors transform translate-y-2 group-hover:translate-y-0 duration-300"
                                >
                                    立即使用
                                </button>
                            </div>
                        </div>

                        <!-- Info -->
                        <div class="p-3 flex flex-col gap-2 flex-1">
                            <div class="flex justify-between items-start gap-2">
                                <h3 class="font-bold text-gray-800 line-clamp-1" :title="item.title">{{ item.title }}</h3>
                                <button 
                                    @click="savePrompt(item)"
                                    class="text-gray-400 hover:text-yellow-500 transition-colors"
                                    title="收藏到预设库"
                                >
                                    ⭐
                                </button>
                            </div>
                            <p class="text-xs text-gray-500 line-clamp-2" :title="item.prompt">
                                {{ item.prompt }}
                            </p>
                            
                            <!-- Tags -->
                            <div class="mt-auto pt-2 flex gap-1 flex-wrap">
                                <span class="text-[10px] bg-gray-100 text-gray-500 px-1.5 py-0.5 rounded">
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
import type { StyleTemplate } from '../types'

// Define the shape of the data from the JSON source
interface WarehouseItem {
    title: string
    prompt: string
    preview: string
    category?: string
    mode?: 'generate' | 'edit'
}

const props = defineProps<{
    mode: 'text-to-image' | 'image-to-image'
}>()

const emit = defineEmits<{
    'close': []
    'use-prompt': [prompt: string]
    'save-prompt': [template: StyleTemplate]
}>()

const prompts = ref<WarehouseItem[]>([])
const loading = ref(true)
const error = ref<string | null>(null)
const selectedCategory = ref('all')

const categories = computed(() => {
    const cats = new Set<string>()
    // Only extract categories from prompts that match the current mode
    const modePrompts = prompts.value.filter(p => {
        const targetMode = props.mode === 'text-to-image' ? 'generate' : 'edit'
        return !p.mode || p.mode === targetMode
    })
    
    modePrompts.forEach(p => {
        if (p.category) cats.add(p.category)
    })
    return Array.from(cats).sort()
})

const filteredPrompts = computed(() => {
    const targetMode = props.mode === 'text-to-image' ? 'generate' : 'edit'
    
    return prompts.value.filter(item => {
        // 1. Filter by Mode
        const modeMatch = !item.mode || item.mode === targetMode
        if (!modeMatch) return false

        // 2. Filter by Category
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
        mode: props.mode // Save with current mode
    }
    emit('save-prompt', template)
    alert(`已收藏 "${item.title}" 到预设库！`)
}

onMounted(() => {
    fetchPrompts()
})
</script>

<style scoped>
.animate-fade-in {
    animation: fadeIn 0.2s ease-out;
}

@keyframes fadeIn {
    from { opacity: 0; transform: scale(0.95); }
    to { opacity: 1; transform: scale(1); }
}
</style>
