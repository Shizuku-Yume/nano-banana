<script setup lang="ts">
import { ref } from 'vue';
import { Plus, X } from 'lucide-vue-next';

const props = withDefaults(defineProps<{
  modelValue: string[];
  max?: number;
}>(), {
  max: 4
});

const emit = defineEmits<{
  (e: 'update:modelValue', images: string[]): void;
}>();

const fileInput = ref<HTMLInputElement | null>(null);

const triggerFileInput = () => {
  fileInput.value?.click();
};

const handleFileSelect = (event: Event) => {
  const target = event.target as HTMLInputElement;
  if (!target.files?.length) return;

  const files = Array.from(target.files);
  const remainingSlots = props.max - props.modelValue.length;
  const filesToProcess = files.slice(0, remainingSlots);

  filesToProcess.forEach(file => {
    const reader = new FileReader();
    reader.onload = (e) => {
      if (e.target?.result && typeof e.target.result === 'string') {
        emit('update:modelValue', [...props.modelValue, e.target.result]);
      }
    };
    reader.readAsDataURL(file);
  });

  target.value = '';
};

const removeImage = (index: number) => {
  const newImages = [...props.modelValue];
  newImages.splice(index, 1);
  emit('update:modelValue', newImages);
};
</script>

<template>
  <div class="flex gap-2 flex-shrink-0">
    <div 
      v-for="(img, index) in modelValue" 
      :key="index"
      class="relative w-12 h-12 rounded-neo overflow-hidden border border-zinc-200 group"
    >
      <img :src="img" class="w-full h-full object-cover" alt="Reference" />
      <button 
        @click="removeImage(index)"
        class="absolute top-0.5 right-0.5 w-5 h-5 bg-red-500 text-white rounded-full text-xs flex items-center justify-center opacity-0 group-hover:opacity-100 transition-opacity cursor-pointer"
      >
        <X :size="12" />
      </button>
    </div>

    <button
      v-if="modelValue.length < max"
      @click="triggerFileInput"
      class="w-12 h-12 border-2 border-dashed border-zinc-300 rounded-neo flex items-center justify-center text-zinc-400 hover:border-brand hover:text-brand transition-colors cursor-pointer"
    >
      <Plus :size="20" />
      <input
        ref="fileInput"
        type="file"
        class="hidden"
        accept="image/*"
        multiple
        @change="handleFileSelect"
      />
    </button>
  </div>
</template>
