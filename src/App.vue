<template>
    <div class="min-h-screen bg-gradient-to-br from-yellow-200 via-yellow-300 to-orange-200 text-gray-900 relative overflow-hidden">
        <!-- 香蕉装饰元素 -->
        <div class="absolute top-10 left-10 text-6xl opacity-20 animate-bounce">🍌</div>
        <div class="absolute top-32 right-20 text-4xl opacity-30 animate-pulse">🍌</div>
        <div class="absolute bottom-20 left-32 text-5xl opacity-25 animate-bounce delay-1000">🍌</div>
        <div class="absolute bottom-40 right-10 text-3xl opacity-20 animate-pulse delay-500">🍌</div>

        <div class="container mx-auto px-3 py-4 relative z-10">
            <!-- Header -->
            <div class="relative mb-6">
                <div class="bg-gradient-to-r from-orange-400 to-yellow-500 rounded-lg p-6 border-4 border-black shadow-lg">
                    <div class="text-center">
                        <h1 class="text-4xl font-black text-white mb-1 flex items-center justify-center gap-2">
                            🍌 Nano<br />
                            <span class="text-yellow-100 text-5xl">Banana</span>
                        </h1>
                        <p class="text-white text-base font-medium">上传你的图片，我来创造艺术！</p>
                    </div>
                </div>
            </div>

            <!-- API设置区域 -->
            <div class="mb-6">
                <div class="flex justify-center">
                    <button
                        @click="showApiSettings = !showApiSettings"
                        :class="[
                            'px-6 py-3 rounded-lg border-4 border-black font-bold text-sm transition-all flex items-center gap-2 shadow-lg',
                            apiKey ? 'bg-green-400 text-white hover:bg-green-500' : 'bg-red-400 text-white hover:bg-red-500 animate-pulse'
                        ]"
                    >
                        <span>🔑</span>
                        <span v-if="!apiKey">需要配置API密钥</span>
                        <span v-else>API密钥已配置</span>
                        <svg :class="['w-4 h-4 transition-transform', showApiSettings ? 'rotate-180' : '']" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7" />
                        </svg>
                    </button>
                </div>

                <!-- API设置折叠面板 -->
                <div v-if="showApiSettings" class="mt-4 max-w-2xl mx-auto">
                    <ApiKeyInput
                        v-model="apiKey"
                        v-model:endpoint="apiEndpoint"
                        v-model:model="selectedModel"
                        :models="modelOptions"
                        :model-loading="isFetchingModels"
                        :model-error="modelsError"
                        :providers="apiConfigs"
                        :active-provider-id="activeProviderId"
                        @fetch-models="handleFetchModels"
                        @model-picked="handleModelPicked"
                        @add-provider="handleAddProvider"
                        @delete-provider="handleDeleteProvider"
                        @switch-provider="handleSwitchProvider"
                        @update-provider-name="handleUpdateProviderName"
                    />
                </div>
            </div>

            <!-- Mode Switcher -->
            <div class="flex justify-center mb-8">
                <div class="bg-white/80 backdrop-blur-sm p-1.5 rounded-xl border-2 border-black shadow-lg inline-flex gap-2">
                    <button
                        @click="generationMode = 'image-to-image'"
                        :class="[
                            'px-6 py-2.5 rounded-lg font-bold transition-all flex items-center gap-2',
                            generationMode === 'image-to-image'
                                ? 'bg-gradient-to-r from-green-400 to-blue-500 text-white shadow-md transform scale-105'
                                : 'text-gray-600 hover:bg-gray-100'
                        ]"
                    >
                        🖼️ 图文生图
                    </button>
                    <button
                        @click="generationMode = 'text-to-image'"
                        :class="[
                            'px-6 py-2.5 rounded-lg font-bold transition-all flex items-center gap-2',
                            generationMode === 'text-to-image'
                                ? 'bg-gradient-to-r from-purple-500 to-pink-500 text-white shadow-md transform scale-105'
                                : 'text-gray-600 hover:bg-gray-100'
                        ]"
                    >
                        ✨ 文生图
                    </button>
                </div>
            </div>

            <!-- Main Content Area -->
            <div class="flex flex-col gap-6">
                
                <!-- Image-to-Image Layout -->
                <div v-if="generationMode === 'image-to-image'" class="grid lg:grid-cols-2 gap-6 items-start animate-fade-in">
                    <!-- Left: Upload -->
                    <div class="flex flex-col h-full">
                        <div class="bg-pink-400 text-white font-bold px-4 py-2 rounded-t-lg border-4 border-black border-b-0 flex items-center gap-2">
                            🍌 上传参考图
                        </div>
                        <div class="flex-1">
                            <ImageUpload v-model="selectedImages" />
                        </div>
                    </div>

                    <!-- Right: Prompt & Settings -->
                    <div class="flex flex-col gap-6">
                        <div class="flex flex-col h-full">
                            <div class="bg-gradient-to-r from-green-400 to-blue-500 text-white font-bold px-4 py-2 rounded-t-lg border-4 border-black border-b-0 flex items-center gap-2">
                                🎨 提示词与风格
                            </div>
                            <div class="flex-1">
                                <StylePromptSelector 
                                    v-model:selectedStyle="selectedStyle" 
                                    v-model:customPrompt="customPrompt" 
                                    :templates="styleTemplates"
                                    :user-templates="userTemplates"
                                    @save-template="handleSaveTemplate"
                                    @delete-template="handleDeleteTemplate"
                                    @import-templates="handleImportTemplates"
                                    @open-warehouse="showWarehouse = true"
                                />
                            </div>
                        </div>

                        <!-- Settings (Conditional) -->
                        <div v-if="showAspectRatioSelector || showGemini3ProConfig" class="flex flex-col gap-4">
                            <div v-if="showAspectRatioSelector" class="bg-white rounded-lg border-4 border-black p-4 shadow-lg">
                                <div class="font-bold mb-2 flex items-center gap-2">📐 图像设置</div>
                                <AspectRatioSelector 
                                    v-model="selectedAspectRatio" 
                                    :model-type="showGemini3ProConfig ? 'gemini-3-pro-image' : 'default'" 
                                    :image-size="gemini3ImageSize" 
                                />
                            </div>
                            
                            <div v-if="showGemini3ProConfig" class="bg-white rounded-lg border-4 border-black p-4 shadow-lg">
                                <div class="font-bold mb-2 flex items-center gap-2">🚀 高级配置</div>
                                <Gemini3ProConfig
                                    v-model:imageSize="gemini3ImageSize"
                                    v-model:enableGoogleSearch="gemini3EnableGoogleSearch"
                                />
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Text-to-Image Layout -->
                <div v-else class="max-w-4xl mx-auto w-full animate-fade-in">
                    <div class="flex flex-col gap-6">
                        <div class="flex flex-col h-full">
                            <div class="bg-gradient-to-r from-purple-500 to-pink-500 text-white font-bold px-4 py-2 rounded-t-lg border-4 border-black border-b-0 flex items-center gap-2">
                                ✨ 创意描述
                            </div>
                            <div class="flex-1">
                                <StylePromptSelector 
                                    v-model:selectedStyle="selectedStyle" 
                                    v-model:customPrompt="customPrompt" 
                                    :templates="styleTemplates"
                                    :user-templates="userTemplates"
                                    @save-template="handleSaveTemplate"
                                    @delete-template="handleDeleteTemplate"
                                    @import-templates="handleImportTemplates"
                                    @open-warehouse="showWarehouse = true"
                                />
                            </div>
                        </div>

                        <!-- Settings (Conditional) -->
                        <div v-if="showAspectRatioSelector || showGemini3ProConfig" class="grid sm:grid-cols-2 gap-4">
                            <div v-if="showAspectRatioSelector" class="bg-white rounded-lg border-4 border-black p-4 shadow-lg">
                                <div class="font-bold mb-2 flex items-center gap-2">📐 图像设置</div>
                                <AspectRatioSelector 
                                    v-model="selectedAspectRatio" 
                                    :model-type="showGemini3ProConfig ? 'gemini-3-pro-image' : 'default'" 
                                    :image-size="gemini3ImageSize" 
                                />
                            </div>
                            
                            <div v-if="showGemini3ProConfig" class="bg-white rounded-lg border-4 border-black p-4 shadow-lg">
                                <div class="font-bold mb-2 flex items-center gap-2">🚀 高级配置</div>
                                <Gemini3ProConfig
                                    v-model:imageSize="gemini3ImageSize"
                                    v-model:enableGoogleSearch="gemini3EnableGoogleSearch"
                                />
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Generate Button -->
                <div class="flex justify-center mt-4">
                    <button
                        @click="handleUnifiedGenerate"
                        :disabled="!canUnifiedGenerate"
                        :class="[
                            'w-full md:w-auto md:min-w-[300px] px-8 py-4 rounded-xl font-black text-xl text-white transition-all duration-200 flex items-center justify-center gap-3 border-4 border-black shadow-[4px_4px_0px_0px_rgba(0,0,0,1)] hover:shadow-[2px_2px_0px_0px_rgba(0,0,0,1)] hover:translate-x-[2px] hover:translate-y-[2px]',
                            canUnifiedGenerate
                                ? generationMode === 'image-to-image' 
                                    ? 'bg-gradient-to-r from-green-400 to-blue-500 hover:brightness-110'
                                    : 'bg-gradient-to-r from-purple-500 to-pink-500 hover:brightness-110'
                                : 'bg-gray-400 cursor-not-allowed shadow-none transform-none'
                        ]"
                    >
                        <span v-if="!isLoading" class="flex items-center gap-2">
                            {{ generationMode === 'image-to-image' ? '🍌 开始施法 (图生图)' : '✨ 开始施法 (文生图)' }}
                        </span>
                        <span v-else class="flex items-center gap-2">
                            🍌 正在施法...
                        </span>
                        <div v-if="isLoading" class="w-8 h-8 border-4 border-white/30 border-t-white rounded-full animate-spin ml-2" />
                    </button>
                </div>
            </div>

            <!-- 生成结果区域：全宽 -->
            <div class="w-full">
                <div class="bg-black text-white font-bold px-4 py-2 rounded-t-lg border-4 border-black border-b-0 flex items-center gap-2">✨ 生成结果</div>
                <ResultDisplay
                    :result="result"
                    :loading="isLoading"
                    :error="error"
                    :can-push="!!result"
                    @download="handleDownloadResult"
                    @push="handlePushDisplayResult"
                />
            </div>

            <!-- Footer -->
            <Footer />

            <PromptWarehouse 
                v-if="showWarehouse"
                :mode="generationMode"
                @close="showWarehouse = false"
                @use-prompt="handleUseWarehousePrompt"
                @save-prompt="handleSaveTemplate"
            />
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, watch } from 'vue'
import ApiKeyInput from './components/ApiKeyInput.vue'
import ImageUpload from './components/ImageUpload.vue'
import StylePromptSelector from './components/StylePromptSelector.vue'
import ResultDisplay from './components/ResultDisplay.vue'
import Footer from './components/Footer.vue'
import AspectRatioSelector from './components/AspectRatioSelector.vue'
import Gemini3ProConfig from './components/Gemini3ProConfig.vue'
import PromptWarehouse from './components/PromptWarehouse.vue'
import { fetchModels, generateImage } from './services/api'
import { styleTemplates } from './data/templates'
import { LocalStorage } from './utils/storage'
import type { ApiModel, GenerateRequest, ModelOption, StyleTemplate, ApiProviderConfig } from './types'
import { DEFAULT_API_ENDPOINT, DEFAULT_MODEL_ID } from './config/api'

// --- State: API Config ---
const apiConfigs = ref<ApiProviderConfig[]>([])
const activeProviderId = ref('')
const showApiSettings = ref(false)
const isFetchingModels = ref(false)
const modelsError = ref<string | null>(null)
const modelOptions = ref<ModelOption[]>([])

// Computed properties for active provider
const activeProvider = computed(() => 
    apiConfigs.value.find(p => p.id === activeProviderId.value) || 
    { apiKey: '', endpoint: '', model: '', id: '', name: '' }
)

const apiKey = computed({
    get: () => activeProvider.value.apiKey,
    set: (val) => updateActiveProvider({ apiKey: val })
})

const apiEndpoint = computed({
    get: () => activeProvider.value.endpoint,
    set: (val) => updateActiveProvider({ endpoint: val })
})

const selectedModel = computed({
    get: () => activeProvider.value.model,
    set: (val) => updateActiveProvider({ model: val })
})

// --- State: Prompts & Styles ---
const userTemplates = ref<StyleTemplate[]>([])
const selectedStyle = ref('')
const customPrompt = ref('')
const showWarehouse = ref(false)
const generationMode = ref<'image-to-image' | 'text-to-image'>('image-to-image')

// --- State: Generation ---
const selectedImages = ref<string[]>([])
const isLoading = ref(false)
const result = ref<string | null>(null)
const error = ref<string | null>(null)

// --- State: Model Specific ---
const selectedAspectRatio = ref('1:1')
const gemini3ImageSize = ref('2K')
const gemini3EnableGoogleSearch = ref(false)

// ... (Lifecycle & Initialization remains same) ...

// ... (API Config Management remains same) ...

// ... (Custom Prompts methods remain same) ...

// ... (Model methods remain same) ...

// --- Methods: Generation ---
const pushImageToUpload = (image: string | null) => {
    if (!image) return
    const filtered = selectedImages.value.filter(existing => existing !== image)
    selectedImages.value = [image, ...filtered]
}

const canUnifiedGenerate = computed(() => {
    const basicChecks = apiKey.value.trim() && 
                       apiEndpoint.value.trim() && 
                       selectedModel.value.trim() && 
                       (selectedStyle.value || customPrompt.value.trim()) &&
                       !isLoading.value

    if (generationMode.value === 'image-to-image') {
        return basicChecks && selectedImages.value.length > 0
    } else {
        return basicChecks
    }
})

const showAspectRatioSelector = computed(() => {
    const modelId = selectedModel.value.toLowerCase().trim()
    if (!modelId) return false

    const segments = modelId.split('/')
    const normalizedId = segments[segments.length - 1]
    return normalizedId === 'gemini-2.5-flash-image' ||
           normalizedId === 'gemini-2.5-flash-image-preview' ||
           modelId.includes('gemini-3-pro-image')
})

const showGemini3ProConfig = computed(() => {
    const modelId = selectedModel.value.toLowerCase().trim()
    if (!modelId) return false
    return modelId.includes('gemini-3-pro-image')
})

const currentModeTemplates = computed(() => {
    return styleTemplates.filter(t => {
        if (generationMode.value === 'text-to-image') {
            return t.mode === 'text-to-image'
        } else {
            // Default to image-to-image or undefined (legacy)
            return t.mode === 'image-to-image' || !t.mode
        }
    })
})

const handleUnifiedGenerate = async () => {
    if (!canUnifiedGenerate.value) return

    isLoading.value = true
    error.value = null
    result.value = null

    try {
        // Use selected style prompt or custom prompt
        const allTemplates = [...styleTemplates, ...userTemplates.value]
        const prompt = selectedStyle.value ? allTemplates.find(t => t.id === selectedStyle.value)?.prompt || customPrompt.value : customPrompt.value

        const request: GenerateRequest = {
            prompt,
            images: generationMode.value === 'image-to-image' ? selectedImages.value : [],
            apikey: apiKey.value,
            endpoint: apiEndpoint.value.trim() || DEFAULT_API_ENDPOINT,
            model: selectedModel.value.trim() || DEFAULT_MODEL_ID
        }

        if (showAspectRatioSelector.value) {
            request.aspectRatio = selectedAspectRatio.value
        }

        if (showGemini3ProConfig.value) {
            request.imageSize = gemini3ImageSize.value
            request.enableGoogleSearch = gemini3EnableGoogleSearch.value
        }

        const response = await generateImage(request)
        result.value = response.imageUrl
    } catch (err) {
        error.value = err instanceof Error ? err.message : '生成失败'
        result.value = null
    } finally {
        isLoading.value = false
    }
}

const handlePushDisplayResult = () => {
    pushImageToUpload(result.value)
}

const handleDownloadResult = async () => {
    const image = result.value
    if (!image) return
    if (typeof window === 'undefined') return

    let downloadUrl = image
    let revokeUrl: string | null = null

    try {
        if (!image.startsWith('data:')) {
            const response = await fetch(image)
            const blob = await response.blob()
            downloadUrl = URL.createObjectURL(blob)
            revokeUrl = downloadUrl
        }

        const link = document.createElement('a')
        const dataMatch = image.match(/^data:image\/([a-zA-Z0-9+]+);/)
        const extension = dataMatch ? dataMatch[1] : 'png'

        link.href = downloadUrl
        link.download = `nano-banana-${Date.now()}.${extension}`
        link.rel = 'noopener'
        document.body.appendChild(link)
        link.click()
        document.body.removeChild(link)

        if (revokeUrl) {
            URL.revokeObjectURL(revokeUrl)
        }
    } catch (downloadError) {
        window.open(image, '_blank', 'noopener')
    }
}
</script>
