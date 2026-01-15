<script setup lang="ts">
import { ref, watch } from 'vue'
import { ImagePlus, X } from 'lucide-vue-next'
import { StylePreset } from '../types'
import Modal from './ui/Modal.vue'
import Button from './ui/Button.vue'

const props = defineProps<{
  preset?: StylePreset
}>()

const emit = defineEmits<{
  (e: 'close'): void
  (e: 'save', preset: Omit<StylePreset, 'id'>): void
  (e: 'delete', id: number): void
}>()

const name = ref('')
const description = ref('')
const thumbnailImage = ref<string | null>(null)
const error = ref('')

watch(() => props.preset, (newPreset) => {
  if (newPreset) {
    name.value = newPreset.name
    description.value = newPreset.description
    thumbnailImage.value = newPreset.referenceImages?.[0] || null
  } else {
    name.value = ''
    description.value = ''
    thumbnailImage.value = null
  }
  error.value = ''
}, { immediate: true })

const handleSave = () => {
  error.value = ''
  const trimmedName = name.value.trim()
  
  if (!trimmedName) {
    error.value = 'Name is required'
    return
  }

  emit('save', {
    name: trimmedName,
    description: description.value.trim(),
    referenceImages: thumbnailImage.value ? [thumbnailImage.value] : undefined
  })
}

const handleDelete = () => {
  if (props.preset?.id && confirm('Delete this preset?')) {
    emit('delete', props.preset.id)
    emit('close')
  }
}

const handleImageUpload = (e: Event) => {
  const input = e.target as HTMLInputElement
  const file = input.files?.[0]
  if (!file) return

  const reader = new FileReader()
  reader.onload = (event) => {
    thumbnailImage.value = event.target?.result as string
  }
  reader.readAsDataURL(file)
  input.value = ''
}

const removeImage = () => {
  thumbnailImage.value = null
}
</script>

<template>
  <Modal :open="true" :title="preset ? 'Edit Style Preset' : 'New Style Preset'" @close="$emit('close')">
    <div class="space-y-5">
      <!-- Name Input -->
      <div>
        <label class="text-sm font-medium text-zinc-700 block mb-1.5">Name</label>
        <input
          v-model="name"
          type="text"
          placeholder="e.g. Neon Cyberpunk"
          class="w-full bg-zinc-100 rounded-neo px-4 py-2.5 outline-none transition-all focus:bg-white focus:ring-2 focus:ring-teal-200 text-zinc-900 placeholder:text-zinc-400"
        />
        <p v-if="error" class="text-xs text-red-500 mt-1">{{ error }}</p>
      </div>

      <!-- Prompt Prefix -->
      <div>
        <label class="text-sm font-medium text-zinc-700 block mb-1.5">Prompt Prefix</label>
        <textarea
          v-model="description"
          rows="3"
          class="w-full bg-zinc-100 rounded-neo px-4 py-2.5 outline-none transition-all focus:bg-white focus:ring-2 focus:ring-teal-200 text-zinc-900 placeholder:text-zinc-400 resize-none"
          placeholder="Enter the style prompt that will be prepended..."
        ></textarea>
        <p class="text-xs text-zinc-500 mt-1">
          This text will be automatically added to the start of your prompts.
        </p>
      </div>

      <!-- Thumbnail Image (Optional, only 1) -->
      <div>
        <label class="text-sm font-medium text-zinc-700 block mb-1.5">
          Thumbnail Image <span class="text-zinc-400 font-normal">(Optional)</span>
        </label>
        
        <div class="flex items-start gap-3">
          <!-- Image Preview or Upload Button -->
          <div v-if="thumbnailImage" class="relative group">
            <img 
              :src="thumbnailImage" 
              alt="Thumbnail"
              class="w-20 h-20 object-cover rounded-lg border border-zinc-200"
            />
            <button
              @click="removeImage"
              class="absolute -top-2 -right-2 w-6 h-6 bg-red-500 text-white rounded-full flex items-center justify-center opacity-0 group-hover:opacity-100 transition-opacity shadow-md"
            >
              <X class="w-3 h-3" />
            </button>
          </div>
          
          <label v-else class="w-20 h-20 border-2 border-dashed border-zinc-300 rounded-lg flex flex-col items-center justify-center cursor-pointer hover:border-teal-400 hover:bg-teal-50 transition-colors">
            <ImagePlus class="w-6 h-6 text-zinc-400" />
            <span class="text-xs text-zinc-400 mt-1">Add</span>
            <input 
              type="file" 
              accept="image/*" 
              class="hidden" 
              @change="handleImageUpload"
            />
          </label>
          
          <p class="text-xs text-zinc-500 flex-1">
            Add a thumbnail to help identify this preset visually.
          </p>
        </div>
      </div>
    </div>

    <!-- Actions -->
    <div class="flex justify-between items-center mt-6 pt-4 border-t border-zinc-100">
      <div>
        <Button
          v-if="preset"
          variant="danger"
          size="sm"
          @click="handleDelete"
        >
          Delete
        </Button>
      </div>
      <div class="flex gap-3">
        <Button variant="secondary" @click="$emit('close')">Cancel</Button>
        <Button variant="primary" @click="handleSave">Save Preset</Button>
      </div>
    </div>
  </Modal>
</template>
