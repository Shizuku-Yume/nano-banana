<script setup lang="ts">
import { Sun, Moon, Monitor } from 'lucide-vue-next'
import { useTheme, type ThemeMode } from '../composables/useTheme'

const { themeMode, setTheme } = useTheme()

const themes: { value: ThemeMode; icon: typeof Sun; label: string }[] = [
    { value: 'system', icon: Monitor, label: '跟随系统' },
    { value: 'light', icon: Sun, label: '浅色' },
    { value: 'dark', icon: Moon, label: '深色' },
]
</script>

<template>
    <div class="flex bg-zinc-100 dark:bg-zinc-800 rounded-full p-1 gap-0.5">
        <button
            v-for="theme in themes"
            :key="theme.value"
            @click="setTheme(theme.value)"
            class="p-2 rounded-full transition-all duration-200"
            :class="[
                themeMode === theme.value
                    ? 'bg-white dark:bg-zinc-700 text-brand dark:text-teal-400 shadow-sm'
                    : 'text-zinc-500 dark:text-zinc-400 hover:text-zinc-700 dark:hover:text-zinc-200'
            ]"
            :title="theme.label"
        >
            <component :is="theme.icon" class="w-4 h-4" />
        </button>
    </div>
</template>
