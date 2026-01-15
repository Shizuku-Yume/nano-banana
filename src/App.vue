<template>
    <div class="min-h-screen bg-zinc-50 text-zinc-900">
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
                        <h2 class="text-2xl font-bold text-zinc-700 mb-2">Nano Banana v2.0</h2>
                        <p class="text-zinc-500">AI Image Generation Suite</p>
                        <p class="text-zinc-400 text-sm mt-4">Use the command center below to start generating</p>
                    </div>
                </div>

                <div v-else-if="activeTab === 'gallery'">
                    <GalleryGrid 
                        :images="galleryImages"
                        :loading="isLoadingGallery"
                        @open-lightbox="openLightbox"
                        @toggle-favorite="handleToggleFavorite"
                        @delete-image="handleDeleteImage"
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
                    />
                    
                    <div v-if="favoriteImages.length === 0 && !isLoadingFavorites" class="text-center py-20">
                        <div class="text-5xl mb-4">❤️</div>
                        <p class="text-zinc-500">No favorites yet</p>
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
                />
            </template>
            
            <template #styles>
                <StyleChips
                    :presets="stylePresets"
                    :selected-id="selectedStyleId"
                    @select="selectedStyleId = $event"
                    @create="showStyleModal = true"
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
            @close="showStyleModal = false"
            @save="handleSavePreset"
        />

        <Toast :toasts="toasts" />
    </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, shallowRef } from 'vue'
import AppHeader from './components/AppHeader.vue'
import CommandCenter from './components/CommandCenter.vue'
import SettingsTray from './components/SettingsTray.vue'
import StyleChips from './components/StyleChips.vue'
import ReferenceImages from './components/ReferenceImages.vue'
import GenerationTimeline from './components/GenerationTimeline.vue'
import GalleryGrid from './components/GalleryGrid.vue'
import Lightbox from './components/Lightbox.vue'
import ApiConfigModal from './components/ApiConfigModal.vue'
import StylePresetModal from './components/StylePresetModal.vue'
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

const params = ref<GenerationParams>({
    aspectRatios: ['1:1'] as AspectRatio[],
    resolution: '2K' as Resolution,
    count: 1
})

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
        modelOptions.value = models.map(m => ({
            id: m.id,
            label: m.name || m.id.split('/').pop() || m.id,
            description: m.description,
            supportsImages: m.id.toLowerCase().includes('image')
        }))
        LocalStorage.saveModelCache(activeProvider.value.endpoint, modelOptions.value)
    } catch (err) {
        addToast('error', err instanceof Error ? err.message : 'Failed to fetch models')
    } finally {
        isFetchingModels.value = false
    }
}

const MAX_CONCURRENT = 3

const handleGenerate = async () => {
    if (!prompt.value.trim()) {
        addToast('warning', 'Please enter a prompt to generate images')
        return
    }
    
    if (!apiKey.value) {
        addToast('warning', 'Please configure your API key first')
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
        addToast('error', 'Generation queue failed unexpectedly')
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
    try {
        const result = await generateImage({
            prompt: task.prompt,
            images: task.referenceImages || [],
            apikey: apiKey.value,
            endpoint: activeProvider.value.endpoint,
            model: activeProvider.value.model,
            aspectRatio: task.aspectRatio
        })
        
        const image: Omit<GeneratedImage, 'id'> = {
            batchId: task.batchId,
            url: result.imageUrl,
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
    } catch (err) {
        let errorMessage = 'Generation failed'
        if (err instanceof Error) {
            if (err.message.includes('Failed to fetch') || err.message.includes('NetworkError')) {
                errorMessage = 'Network error - check your connection'
            } else if (err.message.includes('401') || err.message.includes('Unauthorized')) {
                errorMessage = 'Invalid API key'
            } else if (err.message.includes('429') || err.message.includes('rate')) {
                errorMessage = 'Rate limited - please wait'
            } else {
                errorMessage = err.message
            }
        }
        task.error = errorMessage
        task.status = 'error'
        addToast('error', `Failed: ${errorMessage}`)
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
    const link = document.createElement('a')
    link.href = image.url
    link.download = `nano-banana-${image.id}-${Date.now()}.png`
    document.body.appendChild(link)
    link.click()
    document.body.removeChild(link)
}

const handleReuse = (image: GeneratedImage) => {
    prompt.value = image.prompt
    if (image.referenceImages) {
        referenceImages.value = [...image.referenceImages]
    }
    selectedStyleId.value = image.styleId || null
    activeTab.value = 'create'
    lightbox.value.isOpen = false
}

const handleSavePreset = async (preset: Omit<StylePreset, 'id'>) => {
    await presetStorage.addPreset(preset)
    stylePresets.value = await presetStorage.getAll()
    showStyleModal.value = false
    addToast('success', 'Style preset saved')
}

const addToast = (type: ToastItem['type'], message: string) => {
    const id = crypto.randomUUID()
    toasts.value.push({ id, type, message, duration: 3000 })
    setTimeout(() => {
        toasts.value = toasts.value.filter(t => t.id !== id)
    }, 3000)
}
</script>
