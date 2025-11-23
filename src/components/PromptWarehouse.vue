<template>
    <div class="fixed inset-0 z-50 flex items-center justify-center bg-black/50 backdrop-blur-sm" @click.self="$emit('close')">
        <div class="bg-white w-[90%] h-[90%] max-w-6xl rounded-xl shadow-2xl flex flex-col overflow-hidden animate-fade-in">
            <!-- Header -->
            <div class="p-4 border-b flex justify-between items-center bg-gray-50">
                <div>
                    <h2 class="text-xl font-bold flex items-center gap-2">
                        🏪 提示词仓库
                        <span class="text-xs font-normal text-gray-500 bg-gray-200 px-2 py-0.5 rounded-full">
                            {{ prompts.length }} 个提示词
                        </span>
                    </h2>
                    <p class="text-xs text-gray-500 mt-1">
                        数据来源: <a href="https://glidea.github.io/banana-prompt-quicker/" target="_blank" class="text-blue-500 hover:underline">Banana Prompt Quicker</a>
                    </p>
                </div>
                <button 
                    @click="$emit('close')"
                    class="p-2 hover:bg-gray-200 rounded-full transition-colors text-gray-500 hover:text-gray-700"
                >
                    ✕
                </button>
            </div>

            <!-- Content -->
            <div class="flex-1 overflow-y-auto p-6 bg-gray-50/50">
                <div v-if="loading" class="flex flex-col items-center justify-center h-full text-gray-500 gap-3">
                    <div class="animate-spin text-4xl">🍌</div>
                    <p>正在搬运香蕉...</p>
                </div>

                <div v-else-if="error" class="flex flex-col items-center justify-center h-full text-red-500 gap-2">
                    <p class="text-2xl">⚠️</p>
                    <p>{{ error }}</p>
                    <button @click="fetchPrompts" class="mt-2 px-4 py-2 bg-white border border-red-200 rounded-lg hover:bg-red-50 text-sm">
                        重试
                    </button>
                </div>

                <div v-else class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4">
                    <div 
                        v-for="item in prompts" 
                        :key="item.zh"
                        class="bg-white rounded-lg border border-gray-200 overflow-hidden hover:shadow-lg transition-all group flex flex-col"
                    >
                        <!-- Image Preview -->
                        <div class="aspect-square bg-gray-100 relative overflow-hidden">
                            <img 
                                :src="item.image" 
                                :alt="item.zh"
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
                                <h3 class="font-bold text-gray-800 line-clamp-1" :title="item.zh">{{ item.zh }}</h3>
                                <button 
                                    @click="savePrompt(item)"
                                    class="text-gray-400 hover:text-yellow-500 transition-colors"
                                    title="收藏到预设库"
                                >
                                    ⭐
                                </button>
                            </div>
                            <p class="text-xs text-gray-500 line-clamp-2" :title="item.en">
                                {{ item.en }}
                            </p>
                            
                            <!-- Tags if any (mockup) -->
                            <div class="mt-auto pt-2 flex gap-1 flex-wrap">
                                <span class="text-[10px] bg-gray-100 text-gray-500 px-1.5 py-0.5 rounded">
                                    {{ item.en.length }} chars
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
import { ref, onMounted } from 'vue'
import type { StyleTemplate } from '../types'

// Define the shape of the data from the JSON source
interface WarehouseItem {
    zh: string
    en: string
    image: string
}

const emit = defineEmits<{
    'close': []
    'use-prompt': [prompt: string]
    'save-prompt': [template: StyleTemplate]
}>()

const prompts = ref<WarehouseItem[]>([])
const loading = ref(true)
const error = ref<string | null>(null)

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
    emit('use-prompt', item.en)
    emit('close')
}

const savePrompt = (item: WarehouseItem) => {
    const template: StyleTemplate = {
        id: `warehouse-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
        title: item.zh,
        prompt: item.en,
        description: '来自提示词仓库',
        image: '' // Don't save the image locally as per requirement
    }
    emit('save-prompt', template)
    alert(`已收藏 "${item.zh}" 到预设库！`)
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
