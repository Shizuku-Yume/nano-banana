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

            <!-- 功能布局 -->
            <div class="grid lg:grid-cols-2 gap-4 lg:gap-6 mb-6 lg:items-start">
                <!-- 灵感工坊 -->
                <div class="flex flex-col h-full gap-4">
                    <div class="flex flex-col h-full">
                        <div class="bg-gradient-to-r from-blue-400 to-purple-500 text-white font-bold px-4 py-2 rounded-t-lg border-4 border-black border-b-0 flex items-center gap-2">
                            ✨ 文生图 · 灵感工坊
                        </div>
                        <div class="bg-white border-4 border-black border-t-0 rounded-b-lg p-5 shadow-lg flex flex-col h-full gap-4">
                            <div class="flex flex-col gap-3 flex-1">
                                <label class="font-bold flex items-center gap-2 text-base">🍌 输入你的创意描述：</label>
                                <textarea
                                    v-model="textToImagePrompt"
                                    placeholder="例如：阳光洒在香蕉形热气球上，漂浮在糖果色的天空中"
                                    class="w-full px-4 py-3 border-2 border-black rounded-lg resize-none focus:outline-none focus:ring-2 focus:ring-purple-500 focus:border-transparent min-h-[160px] flex-1"
                                />
                            </div>

                            <p class="text-sm text-gray-600 font-medium flex items-center gap-2">
                                <span>💡</span>
                                <span>填写描述后，使用下方按钮开始创作，生成的图片会展示在下方结果区，可直接下载或继续改图。</span>
                            </p>
                        </div>
                    </div>

                    <!-- 宽高比选择器（仅当选择 Gemini 2.5 Flash Image 系列模型时显示） -->
                    <div v-if="showAspectRatioSelector" class="flex flex-col">
                        <div class="bg-gradient-to-r from-purple-400 to-pink-500 text-white font-bold px-4 py-2 rounded-t-lg border-4 border-black border-b-0 flex items-center gap-2">
                            📐 图像宽高比
                        </div>
                        <AspectRatioSelector v-model="selectedAspectRatio" :model-type="showGemini3ProConfig ? 'gemini-3-pro-image' : 'default'" :image-size="gemini3ImageSize" />
                    </div>

                    <!-- Gemini 3 Pro Image 配置（仅当选择 Gemini 3 Pro Image 模型时显示） -->
                    <div v-if="showGemini3ProConfig" class="flex flex-col">
                        <div class="bg-gradient-to-r from-indigo-400 to-purple-500 text-white font-bold px-4 py-2 rounded-t-lg border-4 border-black border-b-0 flex items-center gap-2">
                            🚀 Gemini 3 Pro Image 配置
                        </div>
                        <Gemini3ProConfig
                            v-model:imageSize="gemini3ImageSize"
                            v-model:enableGoogleSearch="gemini3EnableGoogleSearch"
                        />
                    </div>
                </div>

                <!-- 图文生图流程 -->
                <div class="flex flex-col gap-4 h-full">
                    <div class="flex flex-col h-full">
                        <div class="bg-pink-400 text-white font-bold px-4 py-2 rounded-t-lg border-4 border-black border-b-0 flex items-center gap-2">🍌 图文生图 · 上传图片</div>
                        <div class="flex-1">
                            <ImageUpload v-model="selectedImages" />
                        </div>
                    </div>

                    <div class="flex flex-col h-full">
                        <div class="bg-gradient-to-r from-green-400 to-blue-500 text-white font-bold px-4 py-2 rounded-t-lg border-4 border-black border-b-0 flex items-center gap-2">
                            🎨 图文生图 · 选择风格或自定义提示词
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
                </div>
            </div>

            <!-- 生成按钮 -->
            <div class="mb-6">
                <div class="flex flex-col gap-4 lg:flex-row lg:gap-6">
                    <button
                        @click="handleTextToImageGenerate"
                        :disabled="!canGenerateTextImage"
                        :class="[
                            'flex-1 px-6 py-4 rounded-lg font-bold text-white text-lg transition-all duration-200 flex items-center justify-center gap-3 border-4 border-black shadow-lg',
                            canGenerateTextImage
                                ? 'bg-gradient-to-r from-purple-500 to-blue-500 hover:from-purple-600 hover:to-blue-600 hover:-translate-y-1 hover:shadow-xl'
                                : 'bg-gray-400 cursor-not-allowed'
                        ]"
                    >
                        <span v-if="!isTextToImageLoading" class="flex items-center gap-2 text-xl">🍌 施展魔法（文生图）</span>
                        <span v-else class="flex items-center gap-2 text-xl">🍌 正在施法...</span>
                        <div v-if="isTextToImageLoading" class="w-8 h-8 border-3 border-white/30 border-t-white rounded-full animate-spin" />
                    </button>
                    <button
                        @click="handleGenerate"
                        :disabled="!canGenerate"
                        :class="[
                            'flex-1 px-6 py-4 rounded-lg font-bold text-white text-lg transition-all duration-200 flex items-center justify-center gap-3 border-4 border-black shadow-lg',
                            canGenerate
                                ? 'bg-gradient-to-r from-orange-400 to-yellow-500 hover:from-orange-500 hover:to-yellow-600 hover:-translate-y-1 hover:shadow-xl'
                                : 'bg-gray-400 cursor-not-allowed'
                        ]"
                    >
                        <span v-if="!isLoading" class="flex items-center gap-2 text-xl">🍌 施展魔法（图文生图）</span>
                        <span v-else class="flex items-center gap-2 text-xl">🍌 正在施法...</span>
                        <div v-if="isLoading" class="w-8 h-8 border-3 border-white/30 border-t-white rounded-full animate-spin" />
                    </button>
                </div>
            </div>

            <!-- 生成结果区域：全宽 -->
            <div class="w-full">
                <div class="bg-black text-white font-bold px-4 py-2 rounded-t-lg border-4 border-black border-b-0 flex items-center gap-2">✨ 生成结果</div>
                <ResultDisplay
                    :result="displayResult"
                    :loading="displayLoading"
                    :error="displayError"
                    :can-push="canPushDisplayResult"
                    @download="handleDownloadResult"
                    @push="handlePushDisplayResult"
                />
            </div>

            <!-- Footer -->
            <Footer />
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
const textToImagePrompt = ref('')
const showWarehouse = ref(false)

// --- State: Generation ---
const selectedImages = ref<string[]>([])
const isLoading = ref(false)
const result = ref<string | null>(null)
const error = ref<string | null>(null)
const textToImageResult = ref<string | null>(null)
const textToImageError = ref<string | null>(null)
const isTextToImageLoading = ref(false)
const latestResultSource = ref<'text' | 'image' | null>(null)

// --- State: Model Specific ---
const selectedAspectRatio = ref('1:1')
const gemini3ImageSize = ref('2K')
const gemini3EnableGoogleSearch = ref(false)

// --- Lifecycle & Initialization ---
onMounted(() => {
    // 1. Load Custom Prompts
    userTemplates.value = LocalStorage.getCustomPrompts()

    // 2. Load API Configs
    const savedConfigs = LocalStorage.getApiConfigs()
    const savedActiveId = LocalStorage.getActiveProviderId()

    if (savedConfigs.length > 0) {
        apiConfigs.value = savedConfigs
        if (savedActiveId && savedConfigs.some(c => c.id === savedActiveId)) {
            activeProviderId.value = savedActiveId
        } else {
            activeProviderId.value = savedConfigs[0].id
        }
    } else {
        // Migration: Check for legacy data
        const legacyKey = LocalStorage.getApiKey()
        const legacyEndpoint = LocalStorage.getApiEndpoint()
        const legacyModel = LocalStorage.getModelId()

        const defaultProvider: ApiProviderConfig = {
            id: 'default',
            name: 'Default (OpenRouter)',
            apiKey: legacyKey || '',
            endpoint: legacyEndpoint || DEFAULT_API_ENDPOINT,
            model: legacyModel || DEFAULT_MODEL_ID
        }

        apiConfigs.value = [defaultProvider]
        activeProviderId.value = defaultProvider.id
        saveConfigs()
    }

    // 3. Restore Model Cache for current endpoint
    restoreModelOptionsFromCache(apiEndpoint.value)
    
    // 4. UI State
    if (!apiKey.value) {
        showApiSettings.value = true
    }
})

// --- Methods: API Config Management ---
const saveConfigs = () => {
    LocalStorage.saveApiConfigs(apiConfigs.value)
    LocalStorage.saveActiveProviderId(activeProviderId.value)
}

const updateActiveProvider = (updates: Partial<ApiProviderConfig>) => {
    const index = apiConfigs.value.findIndex(p => p.id === activeProviderId.value)
    if (index !== -1) {
        const updated = { ...apiConfigs.value[index], ...updates }
        apiConfigs.value[index] = updated
        saveConfigs()
        
        // If endpoint changed, handle cache
        if (updates.endpoint) {
             // Logic to clear/reload cache if needed could go here
             // For now, we rely on manual fetch or simple cache restoration
        }
    }
}

const handleAddProvider = () => {
    const newProvider: ApiProviderConfig = {
        id: `provider-${Date.now()}`,
        name: 'New Provider',
        apiKey: '',
        endpoint: DEFAULT_API_ENDPOINT,
        model: DEFAULT_MODEL_ID
    }
    apiConfigs.value.push(newProvider)
    activeProviderId.value = newProvider.id
    saveConfigs()
}

const handleDeleteProvider = (id: string) => {
    if (apiConfigs.value.length <= 1) return
    
    const index = apiConfigs.value.findIndex(p => p.id === id)
    if (index !== -1) {
        apiConfigs.value.splice(index, 1)
        if (activeProviderId.value === id) {
            activeProviderId.value = apiConfigs.value[0].id
        }
        saveConfigs()
    }
}

const handleSwitchProvider = (id: string) => {
    activeProviderId.value = id
    saveConfigs()
    restoreModelOptionsFromCache(apiEndpoint.value)
}

const handleUpdateProviderName = (id: string, name: string) => {
    const provider = apiConfigs.value.find(p => p.id === id)
    if (provider) {
        provider.name = name
        saveConfigs()
    }
}

// --- Methods: Custom Prompts ---
const handleSaveTemplate = (template: StyleTemplate) => {
    userTemplates.value.push(template)
    LocalStorage.saveCustomPrompts(userTemplates.value)
    // Auto-select the new template
    selectedStyle.value = template.id
}

const handleDeleteTemplate = (id: string) => {
    userTemplates.value = userTemplates.value.filter(t => t.id !== id)
    LocalStorage.saveCustomPrompts(userTemplates.value)
    if (selectedStyle.value === id) {
        selectedStyle.value = ''
    }
}

const handleImportTemplates = (templates: StyleTemplate[]) => {
    // Filter out duplicates based on ID
    const newTemplates = templates.filter(t => !userTemplates.value.some(existing => existing.id === t.id))
    if (newTemplates.length > 0) {
        userTemplates.value = [...userTemplates.value, ...newTemplates]
        LocalStorage.saveCustomPrompts(userTemplates.value)
        alert(`成功导入 ${newTemplates.length} 个预设！`)
    } else {
        alert('没有新的预设被导入（可能全部重复）。')
    }
}

const handleUseWarehousePrompt = (prompt: string) => {
    customPrompt.value = prompt
    // Clear selected style if it was set, as we are now using a custom prompt
    selectedStyle.value = ''
}

// --- Methods: Models ---
const handleFetchModels = async () => {
    if (!apiKey.value.trim() || !apiEndpoint.value.trim()) return

    isFetchingModels.value = true
    modelsError.value = null

    try {
        const rawModels = await fetchModels(apiKey.value, apiEndpoint.value)
        const options = mapModelsToOptions(rawModels)

        if (!options.length) {
            throw new Error('未找到可用模型')
        }

        modelOptions.value = options
        LocalStorage.saveModelCache(apiEndpoint.value, options)

        // Auto-select logic
        const currentModelId = selectedModel.value
        const preferred =
            options.find(option => option.id === currentModelId) ||
            options.find(option => option.id === DEFAULT_MODEL_ID) ||
            options.find(option => option.supportsImages) ||
            options[0]

        selectedModel.value = preferred.id
        ensureSelectedOptionPresent()
    } catch (fetchError) {
        modelsError.value = fetchError instanceof Error ? fetchError.message : '无法获取模型列表'
        modelOptions.value = []
        // Don't reset selected model on error, keep what user typed/selected
    } finally {
        isFetchingModels.value = false
    }
}

const mapModelsToOptions = (models: ApiModel[]): ModelOption[] => {
    const uniqueIds = new Set<string>()
    const options: ModelOption[] = []

    models.forEach(model => {
        if (!model?.id || uniqueIds.has(model.id)) return
        uniqueIds.add(model.id)

        const supportsImages = detectImageSupport(model)
        const label = buildModelLabel(model)
        const description = (typeof model.description === 'string' && model.description.trim()) ||
            (typeof (model as Record<string, unknown>).about === 'string' && String((model as Record<string, unknown>).about).trim()) ||
            ''

        options.push({
            id: model.id,
            label,
            description,
            supportsImages
        })
    })

    return options.sort((a, b) => {
        if (a.supportsImages !== b.supportsImages) {
            return a.supportsImages ? -1 : 1
        }
        return a.label.localeCompare(b.label)
    })
}

const detectImageSupport = (model: ApiModel): boolean => {
    const caps = model.capabilities
    if (caps && typeof caps === 'object') {
        if ((caps as Record<string, unknown>).image === true) return true
        if ((caps as Record<string, unknown>).images === true) return true
        if ((caps as Record<string, unknown>).vision === true) return true
        if ((caps as Record<string, unknown>).multimodal === true) return true
    }

    const tags = (model as Record<string, unknown>).tags
    if (Array.isArray(tags) && tags.some(tag => typeof tag === 'string' && /image|vision|photo|picture|art|draw/i.test(tag))) {
        return true
    }

    return /image|vision|flux|art|picture|photo|illustration/i.test(model.id)
}

const buildModelLabel = (model: ApiModel): string => {
    if (model.name && typeof model.name === 'string' && model.name.trim()) {
        return model.name.trim()
    }
    const segments = model.id.split('/')
    const lastSegment = segments[segments.length - 1]
    return lastSegment || model.id
}

const handleModelPicked = () => {
    if (!selectedModel.value.trim()) return
    modelsError.value = null
    if (!showApiSettings.value) return

    setTimeout(() => {
        if (selectedModel.value.trim()) {
            showApiSettings.value = false
        }
    }, 600)
}

const restoreModelOptionsFromCache = (endpoint: string) => {
    const trimmedEndpoint = endpoint.trim()
    if (!trimmedEndpoint) return

    const cached = LocalStorage.getModelCache(trimmedEndpoint)
    if (!cached.length) return

    modelOptions.value = cached
    ensureSelectedOptionPresent()
}

const ensureSelectedOptionPresent = () => {
    const currentId = selectedModel.value.trim()
    if (!currentId) return

    const exists = modelOptions.value.some(option => option.id === currentId)
    if (!exists) {
        modelOptions.value = [
            ...modelOptions.value,
            {
                id: currentId,
                label: buildFallbackLabel(currentId),
                description: '',
                supportsImages: true
            }
        ]
    }

    modelOptions.value = modelOptions.value.sort((a, b) => {
        if (a.supportsImages !== b.supportsImages) {
            return a.supportsImages ? -1 : 1
        }
        return a.label.localeCompare(b.label)
    })
}

const buildFallbackLabel = (modelId: string): string => {
    const segments = modelId.split('/')
    const lastSegment = segments[segments.length - 1]
    return lastSegment || modelId
}

// --- Methods: Generation ---
const pushImageToUpload = (image: string | null) => {
    if (!image) return
    const filtered = selectedImages.value.filter(existing => existing !== image)
    selectedImages.value = [image, ...filtered]
}

const displayLoading = computed(() => {
    if (latestResultSource.value === 'image') return isLoading.value
    if (latestResultSource.value === 'text') return isTextToImageLoading.value
    return isLoading.value || isTextToImageLoading.value
})

const displayResult = computed(() => {
    if (latestResultSource.value === 'image') return result.value
    if (latestResultSource.value === 'text') return textToImageResult.value
    return result.value || textToImageResult.value
})

const displayError = computed(() => {
    if (latestResultSource.value === 'image') return error.value
    if (latestResultSource.value === 'text') return textToImageError.value
    return error.value || textToImageError.value
})

const canPushDisplayResult = computed(() => Boolean(displayResult.value))

const canGenerateTextImage = computed(
    () =>
        apiKey.value.trim() &&
        apiEndpoint.value.trim() &&
        selectedModel.value.trim() &&
        textToImagePrompt.value.trim() &&
        !isTextToImageLoading.value
)

const canGenerate = computed(
    () =>
        apiKey.value.trim() &&
        apiEndpoint.value.trim() &&
        selectedModel.value.trim() &&
        selectedImages.value.length > 0 &&
        (selectedStyle.value || customPrompt.value.trim()) &&
        !isLoading.value
)

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

const handleTextToImageGenerate = async () => {
    if (!canGenerateTextImage.value) return

    latestResultSource.value = 'text'
    isTextToImageLoading.value = true
    textToImageError.value = null
    textToImageResult.value = null

    try {
        const request: GenerateRequest = {
            prompt: textToImagePrompt.value,
            images: [],
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
        textToImageResult.value = response.imageUrl
        latestResultSource.value = 'text'
    } catch (err) {
        textToImageError.value = err instanceof Error ? err.message : '生成失败'
        textToImageResult.value = null
    } finally {
        isTextToImageLoading.value = false
    }
}

const handlePushDisplayResult = () => {
    pushImageToUpload(displayResult.value)
}

const handleDownloadResult = async () => {
    const image = displayResult.value
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

const handleGenerate = async () => {
    if (!canGenerate.value) return

    latestResultSource.value = 'image'
    isLoading.value = true
    error.value = null
    result.value = null

    try {
        // Use selected style prompt or custom prompt
        // Note: styleTemplates are system templates, userTemplates are user custom ones
        const allTemplates = [...styleTemplates, ...userTemplates.value]
        const prompt = selectedStyle.value ? allTemplates.find(t => t.id === selectedStyle.value)?.prompt || customPrompt.value : customPrompt.value

        const request: GenerateRequest = {
            prompt,
            images: selectedImages.value,
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
        latestResultSource.value = 'image'
    } catch (err) {
        error.value = err instanceof Error ? err.message : '生成失败'
        result.value = null
    } finally {
        isLoading.value = false
    }
}
</script>
