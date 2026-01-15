<script setup lang="ts">
import { ref, computed, watch, nextTick } from 'vue'
import { Plus, Trash2, Key, Globe, Cpu } from 'lucide-vue-next'
import Modal from './ui/Modal.vue'
import Input from './ui/Input.vue'
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

// Initialize local state when active provider changes
watch(() => activeProvider.value, (newProvider) => {
  if (newProvider) {
    localName.value = newProvider.name
    localKey.value = newProvider.apiKey
    localEndpoint.value = newProvider.endpoint
    localModel.value = newProvider.model
  }
}, { immediate: true })

// Debounced update for fields
const updateField = (field: keyof ApiProviderConfig, value: string) => {
  if (!activeProvider.value) return
  emit('updateProvider', activeProvider.value.id, { [field]: value })
}

const handleNameChange = (val: string) => {
  localName.value = val
  updateField('name', val)
}

const handleKeyChange = (val: string) => {
  localKey.value = val
  updateField('apiKey', val)
}

const handleEndpointChange = (val: string) => {
  localEndpoint.value = val
  updateField('endpoint', val)
}

const handleModelChange = (e: Event) => {
  const val = (e.target as HTMLSelectElement).value
  localModel.value = val
  updateField('model', val)
}

const handleAddProvider = () => {
  emit('addProvider')
  // Wait for new provider to be created and selected before scrolling
  nextTick(() => {
    const list = document.querySelector('.provider-list')
    if (list) list.scrollTop = list.scrollHeight
  })
}

const confirmDelete = (id: string) => {
  if (confirm('Are you sure you want to delete this provider?')) {
    emit('deleteProvider', id)
  }
}
</script>

<template>
  <Modal :show="true" title="API Configuration" width="max-w-4xl" @close="$emit('close')">
    <div class="flex flex-col md:flex-row h-[80vh] md:h-[500px] -mx-4 md:-mx-6 -mb-6 mt-2 border-t border-zinc-800 animate-scale-in">
      
      <!-- Provider List (Left Sidebar) -->
      <div class="w-full md:w-64 border-b md:border-b-0 md:border-r border-zinc-800 bg-zinc-900/30 flex flex-col shrink-0">
        <div class="p-3 md:p-4 border-b border-zinc-800/50">
          <h3 class="text-xs font-bold text-zinc-500 uppercase tracking-wider mb-2">Providers</h3>
          <Button 
            variant="outline" 
            class="w-full justify-start text-xs border-dashed border-zinc-700 hover:border-teal-500/50 hover:bg-teal-500/10 hover:text-teal-400 transition-all min-h-[36px]"
            @click="handleAddProvider"
          >
            <Plus class="w-3 h-3 mr-2" />
            Add New Provider
          </Button>
        </div>
        
        <div class="flex-1 overflow-y-auto p-2 space-y-1 provider-list max-h-[150px] md:max-h-none">
          <button
            v-for="provider in providers"
            :key="provider.id"
            @click="$emit('switchProvider', provider.id)"
            class="w-full text-left px-3 py-2.5 rounded-lg text-sm transition-all duration-200 group flex items-center justify-between min-h-[44px] md:min-h-0"
            :class="[
              activeProviderId === provider.id 
                ? 'bg-zinc-800 text-teal-400 shadow-sm border border-zinc-700/50 scale-[1.02]' 
                : 'text-zinc-400 hover:bg-zinc-800/50 hover:text-zinc-200 border border-transparent'
            ]"
          >
            <span class="truncate font-medium">{{ provider.name || 'Untitled' }}</span>
            <div 
              v-if="activeProviderId === provider.id"
              class="w-1.5 h-1.5 rounded-full bg-teal-500 shadow-[0_0_8px_rgba(20,184,166,0.5)] animate-pulse"
            ></div>
          </button>
        </div>
      </div>

      <!-- Config Form (Right Content) -->
      <div class="flex-1 flex flex-col bg-zinc-950/50 h-full overflow-hidden">
        <div v-if="activeProvider" class="flex-1 overflow-y-auto p-4 md:p-8 space-y-6">
          
          <!-- Header -->
          <div class="flex items-center justify-between mb-2">
            <div>
              <h2 class="text-lg font-bold text-zinc-100 flex items-center gap-2">
                <span class="w-2 h-6 bg-teal-500 rounded-sm inline-block"></span>
                {{ localName || 'Configure Provider' }}
              </h2>
              <p class="text-xs text-zinc-500 mt-1 pl-4">Manage API credentials and model settings.</p>
            </div>
            
            <Button 
              v-if="providers.length > 1"
              variant="danger" 
              size="sm"
              class="opacity-60 hover:opacity-100 min-h-[36px] min-w-[36px] flex items-center justify-center"
              @click="confirmDelete(activeProvider.id)"
            >
              <Trash2 class="w-4 h-4" />
            </Button>
          </div>

          <!-- Form Fields -->
          <div class="space-y-5 pb-8 md:pb-0">
            
            <!-- Name -->
            <div class="space-y-1.5">
              <label class="text-xs font-semibold text-zinc-400 ml-1">Provider Name</label>
              <Input 
                v-model="localName" 
                placeholder="e.g. OpenRouter, My Local LLM" 
                @input="handleNameChange"
                class="min-h-[44px] md:min-h-[36px]"
              />
            </div>

            <!-- API Key -->
            <div class="space-y-1.5">
              <label class="text-xs font-semibold text-zinc-400 ml-1 flex items-center gap-1.5">
                <Key class="w-3 h-3" /> API Key
              </label>
              <Input 
                v-model="localKey" 
                type="password" 
                placeholder="sk-..." 
                @input="handleKeyChange"
                class="min-h-[44px] md:min-h-[36px]"
              />
              <p class="text-[10px] text-zinc-600 ml-1">Keys are stored locally in your browser.</p>
            </div>

            <!-- Endpoint -->
            <div class="space-y-1.5">
              <label class="text-xs font-semibold text-zinc-400 ml-1 flex items-center gap-1.5">
                <Globe class="w-3 h-3" /> API Endpoint
              </label>
              <div class="flex gap-2">
                <Input 
                  v-model="localEndpoint" 
                  placeholder="https://openrouter.ai/api/v1" 
                  class="flex-1 font-mono text-xs min-h-[44px] md:min-h-[36px]"
                  @input="handleEndpointChange"
                />
              </div>
            </div>

            <!-- Model Selection -->
            <div class="space-y-1.5 pt-2">
               <div class="flex items-center justify-between">
                <label class="text-xs font-semibold text-zinc-400 ml-1 flex items-center gap-1.5">
                  <Cpu class="w-3 h-3" /> Model
                </label>
                <button 
                  @click="$emit('fetchModels')" 
                  class="p-2 md:p-0 text-[10px] font-medium text-teal-500 hover:text-teal-400 transition-colors uppercase tracking-wide flex items-center gap-1"
                  :disabled="isLoadingModels"
                >
                  <span v-if="isLoadingModels" class="animate-spin">⟳</span>
                  <span v-else>↻</span>
                  {{ isLoadingModels ? 'Fetching...' : 'Refresh List' }}
                </button>
              </div>
              
              <div class="relative">
                <select 
                  :value="localModel"
                  @change="handleModelChange"
                  class="w-full bg-zinc-900 border border-zinc-700 text-zinc-200 text-sm rounded-lg p-2.5 min-h-[44px] md:min-h-[40px] focus:ring-2 focus:ring-teal-500/50 focus:border-teal-500 outline-none appearance-none transition-all"
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
                <div class="absolute inset-y-0 right-0 flex items-center px-2 pointer-events-none text-zinc-500">
                  <svg class="w-4 h-4 fill-current" viewBox="0 0 20 20"><path d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z"/></svg>
                </div>
              </div>
              
              <!-- Manual Model Input (Fallback) -->
              <div class="mt-2">
                <div class="flex items-center gap-2 mb-1">
                  <div class="h-px bg-zinc-800 flex-1"></div>
                  <span class="text-[10px] text-zinc-600 uppercase">Or type manually</span>
                  <div class="h-px bg-zinc-800 flex-1"></div>
                </div>
                <Input 
                  v-model="localModel" 
                  placeholder="vendor/model-name" 
                  class="font-mono text-xs opacity-70 focus:opacity-100 transition-opacity min-h-[44px] md:min-h-[36px]"
                  @input="handleModelChange"
                />
              </div>
            </div>

          </div>
        </div>
        
        <div v-else class="flex-1 flex flex-col items-center justify-center text-zinc-500 p-8 text-center">
          <div class="w-16 h-16 rounded-full bg-zinc-900 flex items-center justify-center mb-4 border border-zinc-800">
            <Cpu class="w-8 h-8 opacity-20" />
          </div>
          <p>Select a provider to configure</p>
        </div>
        
        <!-- Action Bar -->
        <div class="p-4 border-t border-zinc-800 bg-zinc-900/50 flex justify-end gap-3 sticky bottom-0">
          <Button variant="ghost" @click="$emit('close')" class="min-h-[44px] md:min-h-[36px]">Done</Button>
        </div>
      </div>
    </div>
  </Modal>
</template>
