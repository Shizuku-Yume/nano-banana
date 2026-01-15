<script setup lang="ts">
import { ref, watch } from 'vue'
import { StylePreset } from '../types'
import Modal from './ui/Modal.vue'
import Input from './ui/Input.vue'
import Button from './ui/Button.vue'
import ReferenceImages from './ReferenceImages.vue'

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
const referenceImages = ref<string[]>([])
const error = ref('')

watch(() => props.preset, (newPreset) => {
  if (newPreset) {
    name.value = newPreset.name
    description.value = newPreset.description
    referenceImages.value = [...(newPreset.referenceImages || [])]
  } else {
    name.value = ''
    description.value = ''
    referenceImages.value = []
  }
  error.value = ''
}, { immediate: true })

const handleSave = () => {
  if (!name.value.trim()) {
    error.value = 'Name is required'
    return
  }

  emit('save', {
    name: name.value.trim(),
    description: description.value.trim(),
    referenceImages: referenceImages.value
  })
}

const handleDelete = () => {
  if (props.preset?.id) {
    emit('delete', props.preset.id)
  }
}
</script>

<template>
  <Modal :open="true" :title="preset ? 'Edit Style Preset' : 'New Style Preset'" @close="$emit('close')">

    <div class="space-y-6 max-h-[70vh] overflow-y-auto px-1 -mx-1 animate-scale-in">
      <div class="space-y-2">
        <Input
          v-model="name"
          label="Name"
          placeholder="e.g. Neon Cyberpunk"
          auto-focus
          class="min-h-[44px] md:min-h-[40px]"
        />
        <p v-if="error" class="text-xs text-red-500 mt-1">{{ error }}</p>
      </div>

      <div class="space-y-2">
        <label class="text-sm font-medium text-zinc-700 block mb-1">
          Prompt Prefix
        </label>
        <textarea
          v-model="description"
          rows="3"
          class="w-full bg-zinc-100/80 shadow-neo-inset rounded-neo px-4 py-2.5 outline-none transition-all focus:bg-white focus:ring-2 focus:ring-teal-100 text-zinc-900 placeholder:text-zinc-400 resize-none min-h-[80px]"
          placeholder="Enter the style prompt that will be prepended..."
        ></textarea>
        <p class="text-xs text-zinc-500">
          This text will be automatically added to the start of your prompts.
        </p>
      </div>

      <div class="space-y-2">
        <label class="text-sm font-medium text-zinc-700 block mb-2">
          Reference Images
        </label>
        <ReferenceImages
          v-model="referenceImages"
          :max="4"
        />
      </div>
    </div>

    <div class="flex flex-col-reverse md:flex-row justify-between w-full mt-6 gap-3 md:gap-0">
      <div>
        <Button
          v-if="preset"
          variant="danger"
          @click="handleDelete"
          class="w-full md:w-auto min-h-[44px] md:min-h-[36px]"
        >
          Delete
        </Button>
      </div>
      <div class="flex flex-col md:flex-row space-y-3 md:space-y-0 md:space-x-3 w-full md:w-auto">
        <Button
          variant="secondary"
          @click="$emit('close')"
          class="w-full md:w-auto min-h-[44px] md:min-h-[36px]"
        >
          Cancel
        </Button>
        <Button
          variant="primary"
          @click="handleSave"
          class="w-full md:w-auto min-h-[44px] md:min-h-[36px]"
        >
          Save Preset
        </Button>
      </div>
    </div>
  </Modal>
</template>
