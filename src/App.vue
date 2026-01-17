<template>
    <div class="min-h-screen bg-zinc-50 dark:bg-zinc-900 text-zinc-900 dark:text-zinc-100 transition-colors duration-200">
        <AppHeader 
            :active-tab="activeTab"
            :is-connected="!!apiKey"
            @update:active-tab="activeTab = $event"
            @open-api-config="showApiConfig = true"
        />

        <main class="pt-20 pb-48 px-4">
            <div class="max-w-6xl mx-auto">
                <div v-if="activeTab === 'create'">
                    <GenerationTimeline 
                        :batches="activeBatches"
                        @open-lightbox="openLightbox"
                        @toggle-favorite="handleToggleFavorite"
                        @delete-image="handleDeleteImage"
                        @reuse="handleReuse"
                    />
                    
                    <div v-if="activeBatches.length === 0" class="text-center py-20">
                        <div class="text-6xl mb-4">🍌</div>
                        <h2 class="text-2xl font-bold text-zinc-700 dark:text-zinc-300 mb-2">Nano Banana v2.0</h2>
                        <p class="text-zinc-500 dark:text-zinc-400">AI 图像生成套件</p>
                        <p class="text-zinc-400 dark:text-zinc-500 text-sm mt-4">使用下方命令中心开始生成</p>
                    </div>
                </div>

                <div v-else-if="activeTab === 'gallery'">
                <GalleryGrid 
                    :images="galleryImages"
                    :loading="isLoadingGallery"
                    @open-lightbox="openLightbox"
                    @toggle-favorite="handleToggleFavorite"
                    @delete-image="handleDeleteImage"
                    @iterate="handleReuse"
                    @download="handleDownload"
                    @load-more="loadMoreGallery"
                />
                </div>

                <div v-else-if="activeTab === 'favorites'">
                <GalleryGrid 
                    :images="favoriteImages"
                    :loading="isLoadingFavorites"
                    @open-lightbox="openLightbox"
                    @toggle-favorite="handleToggleFavorite"
                    @delete-image="handleDeleteImage"
                    @iterate="handleReuse"
                    @download="handleDownload"
                />
                    
                    <div v-if="favoriteImages.length === 0 && !isLoadingFavorites" class="text-center py-20">
                        <div class="text-5xl mb-4">❤️</div>
                        <p class="text-zinc-500 dark:text-zinc-400">暂无收藏</p>
                    </div>
                </div>
            </div>
        </main>

        <CommandCenter
            :prompt="prompt"
            :is-generating="isGenerating"
            :show-settings="showSettings"
            @update:prompt="prompt = $event"
            @update:show-settings="showSettings = $event"
            @generate="handleGenerate"
        >
            <template #settings>
                <SettingsTray
                    v-model:aspect-ratios="params.aspectRatios"
                    v-model:resolution="params.resolution"
                    v-model:count="params.count"
                    v-model:enable-google-search="enableGoogleSearch"
                />
            </template>
            
            <template #presets>
                <PresetPopover
                    :presets="stylePresets"
                    :selected-id="selectedStyleId"
                    @select="selectedStyleId = $event"
                    @apply="handleApplyPreset"
                    @edit="handleEditPreset"
                    @delete="handleDeletePreset"
                    @create="editingPreset = null; showStyleModal = true"
                    @open-warehouse="showPromptWarehouse = true"
                />
            </template>
            
            <template #references>
                <ReferenceImages
                    v-model="referenceImages"
                    :max="4"
                />
            </template>
        </CommandCenter>

        <Lightbox
            v-if="lightbox.isOpen"
            :images="lightbox.images"
            :current-index="lightbox.currentIndex"
            :is-open="lightbox.isOpen"
            @close="lightbox.isOpen = false"
            @navigate="lightbox.currentIndex = $event"
            @favorite="handleToggleFavorite"
            @download="handleDownload"
            @iterate="handleReuse"
        />

        <ApiConfigModal
            v-if="showApiConfig"
            :providers="apiConfigs"
            :active-provider-id="activeProviderId"
            :models="modelOptions"
            :is-loading-models="isFetchingModels"
            @close="showApiConfig = false"
            @add-provider="handleAddProvider"
            @delete-provider="handleDeleteProvider"
            @switch-provider="handleSwitchProvider"
            @update-provider="handleUpdateProvider"
            @fetch-models="handleFetchModels"
        />

        <StylePresetModal
            v-if="showStyleModal"
            :preset="editingPreset"
            @close="showStyleModal = false; editingPreset = null"
            @save="handleSavePreset"
            @delete="handleDeletePreset"
        />

        <PromptWarehouse
            v-if="showPromptWarehouse"
            @close="showPromptWarehouse = false"
            @use-prompt="handleUseWarehousePrompt"
            @save-prompt="handleSaveWarehousePrompt"
        />

        <Toast :toasts="toasts" />
    </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, shallowRef } from 'vue'
import AppHeader from './components/AppHeader.vue'
import CommandCenter from './components/CommandCenter.vue'
import SettingsTray from './components/SettingsTray.vue'
import PresetPopover from './components/PresetPopover.vue'
import ReferenceImages from './components/ReferenceImages.vue'
import GenerationTimeline from './components/GenerationTimeline.vue'
import GalleryGrid from './components/GalleryGrid.vue'
import Lightbox from './components/Lightbox.vue'
import ApiConfigModal from './components/ApiConfigModal.vue'
import StylePresetModal from './components/StylePresetModal.vue'
import PromptWarehouse from './components/PromptWarehouse.vue'
import Toast from './components/ui/Toast.vue'
import { fetchModels, generateImage } from './services/api'
import { imageStorage, presetStorage } from './services/db'
import { LocalStorage } from './utils/storage'
import { migrateV1ToV2 } from './utils/migration'
import type { 
    TabType, 
    GenerationParams, 
    GenerationTask, 
    GeneratedImage, 
    BatchInfo,
    StylePreset,
    StyleTemplate,
    ApiProviderConfig, 
    ModelOption,
    ToastItem,
    LightboxState,
    AspectRatio,
    Resolution
} from './types'
import { DEFAULT_API_ENDPOINT, DEFAULT_MODEL_ID } from './config/api'

const activeTab = ref<TabType>('create')
const showSettings = ref(false)
const showApiConfig = ref(false)
const showStyleModal = ref(false)
const showPromptWarehouse = ref(false)
const toasts = ref<ToastItem[]>([])

const apiConfigs = ref<ApiProviderConfig[]>([])
const activeProviderId = ref('')
const modelOptions = ref<ModelOption[]>([])
const isFetchingModels = ref(false)

const activeProvider = computed(() => 
    apiConfigs.value.find(p => p.id === activeProviderId.value) || 
    { apiKey: '', endpoint: '', model: '', id: '', name: '' }
)

const apiKey = computed({
    get: () => activeProvider.value.apiKey,
    set: (val) => updateActiveProvider({ apiKey: val })
})

const prompt = ref('')
const referenceImages = ref<string[]>([])
const selectedStyleId = ref<string | null>(null)
const stylePresets = shallowRef<StylePreset[]>([])
const editingPreset = ref<StylePreset | null>(null)

const params = ref<GenerationParams>({
    aspectRatios: ['1:1'] as AspectRatio[],
    resolution: '2K' as Resolution,
    count: 1
})

const enableGoogleSearch = ref(false)

const isGenerating = ref(false)
const activeTasks = ref<GenerationTask[]>([])
const activeBatches = ref<BatchInfo[]>([])

const galleryImages = shallowRef<GeneratedImage[]>([])
const favoriteImages = shallowRef<GeneratedImage[]>([])
const isLoadingGallery = ref(false)
const isLoadingFavorites = ref(false)

const lightbox = ref<LightboxState>({
    isOpen: false,
    images: [],
    currentIndex: 0
})

onMounted(async () => {
    await migrateV1ToV2()
    
    const configs = LocalStorage.getApiConfigs()
    if (configs.length > 0) {
        apiConfigs.value = configs
        const activeId = LocalStorage.getActiveProviderId()
        if (activeId && configs.some(c => c.id === activeId)) {
            activeProviderId.value = activeId
        } else {
            activeProviderId.value = configs[0].id
        }
    } else {
        const defaultProvider: ApiProviderConfig = {
            id: crypto.randomUUID(),
            name: 'Default',
            apiKey: LocalStorage.getApiKey(),
            endpoint: LocalStorage.getApiEndpoint() || DEFAULT_API_ENDPOINT,
            model: LocalStorage.getModelId() || DEFAULT_MODEL_ID
        }
        apiConfigs.value = [defaultProvider]
        activeProviderId.value = defaultProvider.id
        saveApiConfigs()
    }

    stylePresets.value = await presetStorage.getAll()
    await loadGallery()
    await loadFavorites()
})

const saveApiConfigs = () => {
    LocalStorage.saveApiConfigs(apiConfigs.value)
    LocalStorage.saveActiveProviderId(activeProviderId.value)
}

const updateActiveProvider = (updates: Partial<ApiProviderConfig>) => {
    const index = apiConfigs.value.findIndex(p => p.id === activeProviderId.value)
    if (index !== -1) {
        apiConfigs.value[index] = { ...apiConfigs.value[index], ...updates }
        saveApiConfigs()
    }
}

const handleAddProvider = () => {
    const newProvider: ApiProviderConfig = {
        id: crypto.randomUUID(),
        name: 'New Provider',
        apiKey: '',
        endpoint: DEFAULT_API_ENDPOINT,
        model: DEFAULT_MODEL_ID
    }
    apiConfigs.value.push(newProvider)
    activeProviderId.value = newProvider.id
    saveApiConfigs()
}

const handleDeleteProvider = (id: string) => {
    if (apiConfigs.value.length <= 1) return
    apiConfigs.value = apiConfigs.value.filter(p => p.id !== id)
    if (activeProviderId.value === id) {
        activeProviderId.value = apiConfigs.value[0].id
    }
    saveApiConfigs()
}

const handleSwitchProvider = (id: string) => {
    activeProviderId.value = id
    saveApiConfigs()
}

const handleUpdateProvider = (id: string, updates: Partial<ApiProviderConfig>) => {
    const provider = apiConfigs.value.find(p => p.id === id)
    if (provider) {
        Object.assign(provider, updates)
        saveApiConfigs()
    }
}

const handleFetchModels = async () => {
    if (!apiKey.value || !activeProvider.value.endpoint) return
    
    isFetchingModels.value = true
    try {
        const cached = LocalStorage.getModelCache(activeProvider.value.endpoint)
        if (cached.length > 0) {
            modelOptions.value = cached
        }

        const models = await fetchModels(activeProvider.value.endpoint, apiKey.value)
        const mappedModels = models.map(m => {
            const supportsImages = m.id.toLowerCase().includes('image')
            return {
                id: m.id,
                label: supportsImages 
                    ? `🖼️ ${m.name?.trim() ? `${m.id} - ${m.name.trim()}` : m.id}`
                    : (m.name?.trim() ? `${m.id} - ${m.name.trim()}` : m.id),
                description: m.description,
                supportsImages
            }
        })
        // 排序：生图模型在前
        modelOptions.value = mappedModels.sort((a, b) => {
            if (a.supportsImages && !b.supportsImages) return -1
            if (!a.supportsImages && b.supportsImages) return 1
            return 0
        })
        LocalStorage.saveModelCache(activeProvider.value.endpoint, modelOptions.value)
    } catch (err) {
        addToast('error', err instanceof Error ? err.message : '获取模型列表失败')
    } finally {
        isFetchingModels.value = false
    }
}

const MAX_CONCURRENT = 4

const handleGenerate = async () => {
    if (!prompt.value.trim()) {
        addToast('warning', '请输入提示词以生成图像')
        return
    }
    
    if (!apiKey.value) {
        addToast('warning', '请先配置您的 API 密钥')
        showApiConfig.value = true
        return
    }
    
    if (isGenerating.value) return
    
    isGenerating.value = true
    const batchId = crypto.randomUUID()
    const tasks: GenerationTask[] = []
    
    for (const ratio of params.value.aspectRatios) {
        for (let i = 0; i < params.value.count; i++) {
            tasks.push({
                id: crypto.randomUUID(),
                batchId,
                status: 'pending',
                aspectRatio: ratio,
                prompt: prompt.value,
                referenceImages: referenceImages.value.length > 0 ? [...referenceImages.value] : undefined,
                styleId: selectedStyleId.value || undefined,
                resolution: params.value.resolution,
                createdAt: Date.now()
            })
        }
    }
    
    const batchInfo: BatchInfo = {
        batchId,
        prompt: prompt.value,
        styleId: selectedStyleId.value || undefined,
        referenceImages: referenceImages.value.length > 0 ? [...referenceImages.value] : undefined,
        createdAt: Date.now(),
        tasks
    }
    
    activeBatches.value = [batchInfo, ...activeBatches.value]
    activeTasks.value.push(...tasks)
    
    try {
        await processQueue(tasks, MAX_CONCURRENT)
    } catch (err) {
        addToast('error', '生成队列意外失败')
        console.error('Queue processing error:', err)
    } finally {
        isGenerating.value = false
        await loadGallery()
    }
}

async function processQueue(tasks: GenerationTask[], concurrency: number) {
    const executing: Promise<void>[] = []
    
    for (const task of tasks) {
        const promise = executeTask(task).then(() => {
            executing.splice(executing.indexOf(promise), 1)
        })
        executing.push(promise)
        
        if (executing.length >= concurrency) {
            await Promise.race(executing)
        }
    }
    
    await Promise.all(executing)
}

async function executeTask(task: GenerationTask) {
    task.status = 'generating'
    activeBatches.value = [...activeBatches.value]
    try {
        const result = await generateImage({
            prompt: task.prompt,
            images: task.referenceImages || [],
            apikey: apiKey.value,
            endpoint: activeProvider.value.endpoint,
            model: activeProvider.value.model,
            aspectRatio: task.aspectRatio,
            imageSize: task.resolution,
            enableGoogleSearch: enableGoogleSearch.value
        })
        
        const imageUrl = result.imageUrls[0]
        if (!imageUrl) {
            throw new Error('No image returned from API')
        }
        
        const image: Omit<GeneratedImage, 'id'> = {
            batchId: task.batchId,
            url: imageUrl,
            prompt: task.prompt,
            aspectRatio: task.aspectRatio,
            resolution: task.resolution,
            timestamp: Date.now(),
            styleId: task.styleId,
            referenceImages: task.referenceImages,
            isFavorite: false
        }
        
        const id = await imageStorage.addImage(image)
        task.data = { ...image, id }
        task.status = 'success'
        activeBatches.value = [...activeBatches.value]
    } catch (err) {
        let errorMessage = '生成失败'
        if (err instanceof Error) {
            if (err.message.includes('Failed to fetch') || err.message.includes('NetworkError')) {
                errorMessage = '网络错误 - 请检查连接'
            } else if (err.message.includes('401') || err.message.includes('Unauthorized')) {
                errorMessage = 'API 密钥无效'
            } else if (err.message.includes('429') || err.message.includes('rate')) {
                errorMessage = '请求过于频繁 - 请稍候'
            } else {
                errorMessage = err.message
            }
        }
        task.error = errorMessage
        task.status = 'error'
        activeBatches.value = [...activeBatches.value]
        addToast('error', `失败: ${errorMessage}`)
    }
}

const loadGallery = async () => {
    isLoadingGallery.value = true
    try {
        galleryImages.value = await imageStorage.getRecent(50, 0)
    } finally {
        isLoadingGallery.value = false
    }
}

const loadMoreGallery = async () => {
    const more = await imageStorage.getRecent(50, galleryImages.value.length)
    galleryImages.value = [...galleryImages.value, ...more]
}

const loadFavorites = async () => {
    isLoadingFavorites.value = true
    try {
        favoriteImages.value = await imageStorage.getFavorites()
    } finally {
        isLoadingFavorites.value = false
    }
}

const handleToggleFavorite = async (id: number) => {
    await imageStorage.toggleFavorite(id)
    await loadGallery()
    await loadFavorites()
}

const handleDeleteImage = async (id: number) => {
    await imageStorage.deleteImage(id)
    await loadGallery()
    await loadFavorites()
}

const openLightbox = (images: GeneratedImage[], index: number) => {
    lightbox.value = {
        isOpen: true,
        images,
        currentIndex: index
    }
}

const handleDownload = async (image: GeneratedImage) => {
    try {
        const response = await fetch(image.url)
        const blob = await response.blob()
        const blobUrl = URL.createObjectURL(blob)
        const link = document.createElement('a')
        link.href = blobUrl
        link.download = `nano-banana-${image.id}-${Date.now()}.png`
        document.body.appendChild(link)
        link.click()
        document.body.removeChild(link)
        URL.revokeObjectURL(blobUrl)
    } catch (err) {
        addToast('error', '下载失败')
        console.error('Download failed:', err)
    }
}

const handleReuse = (image: GeneratedImage) => {
    prompt.value = image.prompt
    referenceImages.value = [image.url]
    selectedStyleId.value = image.styleId || null
    activeTab.value = 'create'
    lightbox.value.isOpen = false
    addToast('success', '已添加为参考图')
}

const handleSavePreset = async (preset: Omit<StylePreset, 'id'>) => {
    if (editingPreset.value?.id) {
        await presetStorage.update(editingPreset.value.id, preset)
        addToast('success', '预设已更新')
    } else {
        await presetStorage.addPreset(preset)
        addToast('success', '预设已保存')
    }
    stylePresets.value = await presetStorage.getAll()
    showStyleModal.value = false
    editingPreset.value = null
}

const handleApplyPreset = (preset: StylePreset) => {
    if (preset.description) {
        prompt.value = preset.description + (prompt.value ? '\n' + prompt.value : '')
    }
    if (preset.referenceImages?.length) {
        referenceImages.value = [...preset.referenceImages, ...referenceImages.value].slice(0, 4)
    }
    addToast('success', `已应用「${preset.name}」`)
}

const handleEditPreset = (preset: StylePreset) => {
    editingPreset.value = preset
    showStyleModal.value = true
}

const handleDeletePreset = async (id: number) => {
    await presetStorage.delete(id)
    stylePresets.value = await presetStorage.getAll()
    if (selectedStyleId.value === String(id)) {
        selectedStyleId.value = null
    }
    addToast('success', '预设已删除')
}

const handleUseWarehousePrompt = (warehousePrompt: string) => {
    prompt.value = warehousePrompt
    showPromptWarehouse.value = false
    addToast('success', '提示词已加载')
}

const handleSaveWarehousePrompt = async (template: StyleTemplate) => {
    const preset: Omit<StylePreset, 'id'> = {
        name: template.title,
        description: template.prompt
    }
    await presetStorage.addPreset(preset)
    stylePresets.value = await presetStorage.getAll()
    addToast('success', `「${template.title}」已保存到预设`)
}

const addToast = (type: ToastItem['type'], message: string) => {
    const id = crypto.randomUUID()
    toasts.value.push({ id, type, message, duration: 3000 })
    setTimeout(() => {
        toasts.value = toasts.value.filter(t => t.id !== id)
    }, 3000)
}
</script>
