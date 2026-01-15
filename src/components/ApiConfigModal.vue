<script setup lang="ts">
import { ref, computed, watch } from 'vue'
import { Plus, Trash2, Key, Globe, Cpu, X } from 'lucide-vue-next'
import Button from './ui/Button.vue'
import type { ApiProviderConfig, ModelOption } from '../types'

const props = defineProps<{
  providers: ApiProviderConfig[]
  activeProviderId: string
  models: ModelOption[]
  isLoadingModels: boolean
}>()

const emit = defineEmits<{
  (e: 'close'): void
  (e: 'addProvider'): void
  (e: 'deleteProvider', id: string): void
  (e: 'switchProvider', id: string): void
  (e: 'updateProvider', id: string, updates: Partial<ApiProviderConfig>): void
  (e: 'fetchModels'): void
}>()

const activeProvider = computed(() => 
  props.providers.find(p => p.id === props.activeProviderId)
)

const localName = ref('')
const localKey = ref('')
const localEndpoint = ref('')
const localModel = ref('')

watch(() => activeProvider.value, (newProvider) => {
  if (newProvider) {
    localName.value = newProvider.name
    localKey.value = newProvider.apiKey
    localEndpoint.value = newProvider.endpoint
    localModel.value = newProvider.model
  }
}, { immediate: true })

const updateField = (field: keyof ApiProviderConfig, value: string) => {
  if (!activeProvider.value) return
  emit('updateProvider', activeProvider.value.id, { [field]: value })
}

const handleNameBlur = () => updateField('name', localName.value)
const handleKeyBlur = () => updateField('apiKey', localKey.value)
const handleEndpointBlur = () => updateField('endpoint', localEndpoint.value)

const handleModelChange = (e: Event) => {
  const val = (e.target as HTMLSelectElement).value
  localModel.value = val
  updateField('model', val)
}

const handleManualModelBlur = () => updateField('model', localModel.value)

const handleAddProvider = () => {
  emit('addProvider')
}

const confirmDelete = (id: string) => {
  if (confirm('Are you sure you want to delete this provider?')) {
    emit('deleteProvider', id)
  }
}
</script>

<template>
  <div 
    class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-zinc-900/50 backdrop-blur-sm animate-fade-in"
    @click.self="$emit('close')"
  >
    <div class="bg-white rounded-neo-lg shadow-2xl w-full max-w-4xl max-h-[90vh] flex flex-col overflow-hidden">
      <!-- Header -->
      <div class="p-4 border-b border-zinc-200 flex justify-between items-center bg-zinc-50">
        <h2 class="text-xl font-bold text-zinc-800">API Configuration</h2>
        <button 
          @click="$emit('close')"
          class="w-8 h-8 flex items-center justify-center rounded-full hover:bg-zinc-200 transition-colors text-zinc-500"
        >
          <X class="w-5 h-5" />
        </button>
      </div>

      <div class="flex flex-col md:flex-row flex-1 overflow-hidden">
        <!-- Provider List (Left Sidebar) -->
        <div class="w-full md:w-56 border-b md:border-b-0 md:border-r border-zinc-200 bg-zinc-50 flex flex-col shrink-0">
          <div class="p-3 border-b border-zinc-200">
            <Button 
              variant="outline" 
              class="w-full justify-start text-xs border-dashed hover:border-teal-500 hover:bg-teal-50 hover:text-teal-700 transition-all"
              @click="handleAddProvider"
            >
              <Plus class="w-3 h-3 mr-2" />
              Add Provider
            </Button>
          </div>
          
          <div class="flex-1 overflow-y-auto p-2 space-y-1 max-h-[120px] md:max-h-none">
            <button
              v-for="provider in providers"
              :key="provider.id"
              @click="$emit('switchProvider', provider.id)"
              class="w-full text-left px-3 py-2.5 rounded-lg text-sm transition-all duration-200 group flex items-center justify-between"
              :class="[
                activeProviderId === provider.id 
                  ? 'bg-teal-50 text-teal-700 border border-teal-200' 
                  : 'text-zinc-600 hover:bg-zinc-100 border border-transparent'
              ]"
            >
              <span class="truncate font-medium">{{ provider.name || 'Untitled' }}</span>
              <div 
                v-if="activeProviderId === provider.id"
                class="w-2 h-2 rounded-full bg-teal-500"
              ></div>
            </button>
          </div>
        </div>

        <!-- Config Form (Right Content) -->
        <div class="flex-1 flex flex-col overflow-hidden">
          <div v-if="activeProvider" class="flex-1 overflow-y-auto p-4 md:p-6 space-y-5">
            
            <!-- Header with Delete -->
            <div class="flex items-center justify-between">
              <div>
                <h3 class="text-lg font-bold text-zinc-800">{{ localName || 'Configure Provider' }}</h3>
                <p class="text-xs text-zinc-500 mt-0.5">Manage API credentials and model settings</p>
              </div>
              
              <Button 
                v-if="providers.length > 1"
                variant="danger" 
                size="sm"
                class="opacity-60 hover:opacity-100"
                @click="confirmDelete(activeProvider.id)"
              >
                <Trash2 class="w-4 h-4" />
              </Button>
            </div>

            <!-- Form Fields -->
            <div class="space-y-4">
              
              <!-- Name -->
              <div class="space-y-1.5">
                <label class="text-sm font-medium text-zinc-700">Provider Name</label>
                <input 
                  v-model="localName"
                  @blur="handleNameBlur"
                  placeholder="e.g. OpenRouter, My Local LLM"
                  class="w-full bg-zinc-100 rounded-neo px-4 py-2.5 outline-none transition-all focus:bg-white focus:ring-2 focus:ring-teal-200 text-zinc-900 placeholder:text-zinc-400"
                />
              </div>

              <!-- API Key -->
              <div class="space-y-1.5">
                <label class="text-sm font-medium text-zinc-700 flex items-center gap-1.5">
                  <Key class="w-4 h-4 text-zinc-400" /> API Key
                </label>
                <input 
                  v-model="localKey"
                  @blur="handleKeyBlur"
                  type="password"
                  placeholder="sk-..."
                  class="w-full bg-zinc-100 rounded-neo px-4 py-2.5 outline-none transition-all focus:bg-white focus:ring-2 focus:ring-teal-200 text-zinc-900 placeholder:text-zinc-400 font-mono"
                />
                <p class="text-xs text-zinc-500">Keys are stored locally in your browser</p>
              </div>

              <!-- Endpoint -->
              <div class="space-y-1.5">
                <label class="text-sm font-medium text-zinc-700 flex items-center gap-1.5">
                  <Globe class="w-4 h-4 text-zinc-400" /> API Endpoint
                </label>
                <input 
                  v-model="localEndpoint"
                  @blur="handleEndpointBlur"
                  placeholder="https://openrouter.ai/api/v1"
                  class="w-full bg-zinc-100 rounded-neo px-4 py-2.5 outline-none transition-all focus:bg-white focus:ring-2 focus:ring-teal-200 text-zinc-900 placeholder:text-zinc-400 font-mono text-sm"
                />
              </div>

              <!-- Model Selection -->
              <div class="space-y-1.5">
                <div class="flex items-center justify-between">
                  <label class="text-sm font-medium text-zinc-700 flex items-center gap-1.5">
                    <Cpu class="w-4 h-4 text-zinc-400" /> Model
                  </label>
                  <button 
                    @click="$emit('fetchModels')" 
                    class="text-xs font-medium text-teal-600 hover:text-teal-700 transition-colors flex items-center gap-1"
                    :disabled="isLoadingModels"
                  >
                    <span v-if="isLoadingModels" class="animate-spin">⟳</span>
                    <span v-else>↻</span>
                    {{ isLoadingModels ? 'Fetching...' : 'Refresh' }}
                  </button>
                </div>
                
                <select 
                  :value="localModel"
                  @change="handleModelChange"
                  class="w-full bg-zinc-100 border border-zinc-200 text-zinc-900 text-sm rounded-neo p-2.5 focus:ring-2 focus:ring-teal-200 focus:border-teal-300 outline-none appearance-none transition-all"
                >
                  <option value="" disabled>Select a model...</option>
                  <optgroup v-if="models.length > 0" label="Available Models">
                    <option 
                      v-for="model in models" 
                      :key="model.id" 
                      :value="model.id"
                    >
                      {{ model.label }}
                    </option>
                  </optgroup>
                  <option v-else value="google/gemini-2.5-flash-image-preview:free">
                    Default (Gemini 2.5 Flash)
                  </option>
                </select>
                
                <!-- Manual Model Input -->
                <div class="mt-2">
                  <div class="flex items-center gap-2 mb-1">
                    <div class="h-px bg-zinc-200 flex-1"></div>
                    <span class="text-xs text-zinc-400">Or type manually</span>
                    <div class="h-px bg-zinc-200 flex-1"></div>
                  </div>
                  <input 
                    v-model="localModel"
                    @blur="handleManualModelBlur"
                    placeholder="vendor/model-name"
                    class="w-full bg-zinc-100 rounded-neo px-4 py-2 outline-none transition-all focus:bg-white focus:ring-2 focus:ring-teal-200 text-zinc-900 placeholder:text-zinc-400 font-mono text-sm"
                  />
                </div>
              </div>

            </div>
          </div>
          
          <div v-else class="flex-1 flex flex-col items-center justify-center text-zinc-400 p-8 text-center">
            <div class="w-16 h-16 rounded-full bg-zinc-100 flex items-center justify-center mb-4">
              <Cpu class="w-8 h-8 opacity-40" />
            </div>
            <p>Select a provider to configure</p>
          </div>
          
          <!-- Action Bar -->
          <div class="p-4 border-t border-zinc-200 bg-zinc-50 flex justify-end">
            <Button variant="primary" @click="$emit('close')">Done</Button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
