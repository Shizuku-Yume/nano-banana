<template>
    <div class="bg-white border-4 border-black border-t-0 rounded-b-lg p-4 shadow-lg h-full flex flex-col">
        <!-- 顶部操作栏 -->
        <div class="flex justify-between items-center mb-4">
            <div class="text-sm font-bold text-gray-700">
                📝 提示词编辑
            </div>
            <div class="flex gap-2">
                <button
                    @click.stop="$emit('open-warehouse')"
                    class="text-xs bg-purple-100 hover:bg-purple-200 text-purple-700 px-2 py-1 rounded border border-purple-300 transition-colors flex items-center gap-1 font-bold"
                    title="打开提示词仓库"
                >
                    🏪 仓库
                </button>
                <button
                    @click="copyAllPresets"
                    class="text-xs bg-gray-100 hover:bg-gray-200 text-gray-700 px-2 py-1 rounded border border-gray-300 transition-colors flex items-center gap-1"
                    title="复制所有预设到剪贴板"
                >
                    📋 导出
                </button>
                <button
                    @click="importPresets"
                    class="text-xs bg-gray-100 hover:bg-gray-200 text-gray-700 px-2 py-1 rounded border border-gray-300 transition-colors flex items-center gap-1"
                    title="从剪贴板导入预设"
                >
                    📥 导入
                </button>
            </div>
        </div>

        <!-- 主要编辑区域 -->
        <div class="flex flex-col gap-3 flex-1">
            <div class="relative flex-1">
                <textarea
                    :value="customPrompt"
                    @input="updateCustomPrompt(($event.target as HTMLTextAreaElement).value)"
                    placeholder="在此输入提示词，或从下方选择预设..."
                    class="w-full h-full px-4 py-3 border-2 border-black rounded-lg resize-none focus:outline-none focus:ring-2 focus:ring-orange-500 focus:border-transparent min-h-[120px]"
                />
                <button
                    v-if="customPrompt"
                    @click="saveAsPreset"
                    class="absolute bottom-3 right-3 text-xs bg-yellow-400 hover:bg-yellow-500 text-black font-bold px-3 py-1.5 rounded-full shadow-sm transition-colors flex items-center gap-1"
                >
                    💾 存为预设
                </button>
            </div>
        </div>

        <!-- 预设列表区域 -->
        <div class="mt-4">
            <div class="flex items-center justify-between mb-2">
                <label class="text-sm font-bold text-gray-700">📚 预设风格库</label>
                <span class="text-xs text-gray-500">点击应用</span>
            </div>
            
            <div class="grid grid-cols-1 gap-2 max-h-[200px] overflow-y-auto pr-1">
                <div
                    v-for="template in allTemplates"
                    :key="template.id"
                    class="group relative"
                >
                    <div
                        @click="toggleStyle(template)"
                        :class="[
                            'p-3 rounded-lg border-2 cursor-pointer transition-all flex items-center gap-3',
                            selectedStyle === template.id 
                                ? 'bg-yellow-100 border-orange-500' 
                                : 'bg-white border-gray-200 hover:border-orange-300'
                        ]"
                    >
                        <div class="flex-1 min-w-0">
                            <div class="text-sm font-bold truncate">{{ template.title }}</div>
                            <div class="text-xs text-gray-500 truncate">{{ template.description }}</div>
                        </div>
                        <button
                            v-if="isUserTemplate(template.id)"
                            @click.stop="deletePreset(template.id)"
                            class="opacity-0 group-hover:opacity-100 p-1.5 text-red-500 hover:bg-red-50 rounded transition-all"
                            title="删除预设"
                        >
                            🗑️
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import type { StyleTemplate } from '../types'

const props = defineProps<{
    selectedStyle: string
    customPrompt: string
    templates: StyleTemplate[]
    userTemplates: StyleTemplate[]
    mode: 'text-to-image' | 'image-to-image'
}>()

const emit = defineEmits<{
    'update:selectedStyle': [value: string]
    'update:customPrompt': [value: string]
    'save-template': [template: StyleTemplate]
    'delete-template': [id: string]
    'import-templates': [templates: StyleTemplate[]]
    'open-warehouse': []
}>()

const allTemplates = computed(() => [...props.userTemplates, ...props.templates])

const isUserTemplate = (id: string) => {
    return props.userTemplates.some(t => t.id === id)
}

const updateCustomPrompt = (value: string) => {
    // 如果用户修改了内容，且内容与当前选中的预设不一致，则取消选中预设
    if (props.selectedStyle) {
        const currentTemplate = allTemplates.value.find(t => t.id === props.selectedStyle)
        if (currentTemplate && currentTemplate.prompt !== value) {
            emit('update:selectedStyle', '')
        }
    }
    emit('update:customPrompt', value)
}

const toggleStyle = (template: StyleTemplate) => {
    if (props.selectedStyle === template.id) {
        // 取消选中，不清除文本，让用户可以在此基础上修改
        emit('update:selectedStyle', '')
    } else {
        // 选中，应用预设文本
        emit('update:selectedStyle', template.id)
        emit('update:customPrompt', template.prompt)
    }
}

const saveAsPreset = () => {
    const title = prompt('请输入预设名称：', '我的自定义风格')
    if (!title) return

    const description = prompt('请输入预设描述（可选）：', '用户自定义风格') || '用户自定义风格'

    const newTemplate: StyleTemplate = {
        id: `custom-${Date.now()}`,
        title,
        prompt: props.customPrompt,
        description,
        image: '',
        mode: props.mode
    }

    emit('save-template', newTemplate)
}

const deletePreset = (id: string) => {
    if (confirm('确定要删除这个预设吗？')) {
        emit('delete-template', id)
    }
}

const copyAllPresets = async () => {
    // 导出时移除 image 字段
    const exportData = allTemplates.value.map(({ image, ...rest }) => rest)
    try {
        await navigator.clipboard.writeText(JSON.stringify(exportData, null, 2))
        alert('✅ 已将所有预设复制到剪贴板！')
    } catch (err) {
        console.error('复制失败:', err)
        alert('❌ 复制失败，请检查浏览器权限')
    }
}

const importPresets = async () => {
    try {
        const text = await navigator.clipboard.readText()
        if (!text) {
            alert('⚠️ 剪贴板为空')
            return
        }

        let imported: any
        try {
            imported = JSON.parse(text)
        } catch (e) {
            alert('❌ 剪贴板内容不是有效的 JSON 格式')
            return
        }

        if (!Array.isArray(imported)) {
            // 尝试支持单个对象导入
            if (typeof imported === 'object' && imported.id && imported.prompt) {
                imported = [imported]
            } else {
                alert('❌ 格式错误：需要预设数组')
                return
            }
        }

        // 简单的格式验证
        const validTemplates = imported.filter((t: any) => t.id && t.prompt && t.title) as StyleTemplate[]
        
        if (validTemplates.length === 0) {
            alert('❌ 没有找到有效的预设数据')
            return
        }

        if (confirm(`找到 ${validTemplates.length} 个预设，确定要导入吗？\n注意：ID 冲突的预设将被跳过。`)) {
            emit('import-templates', validTemplates)
        }
    } catch (err) {
        console.error('导入失败:', err)
        alert('❌ 导入失败，请检查浏览器权限')
    }
}
</script>
