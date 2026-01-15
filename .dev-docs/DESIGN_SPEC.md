# Nano Banana v2.0 设计规范

> **版本**: 1.0 | **最后更新**: 2026-01-15
> 
> **项目定位**: 基于 Vue 3 + TypeScript + TailwindCSS 的专业级 AI 图像生成应用，融合 Giga Peach 的批量生成能力与 CardForge 的微拟物专业风格。

---

## 1. 设计愿景

### 1.1 核心理念

**从"香蕉主题趣味风格"进化为"微拟物专业风格 (Subtle Neo-Matte Professional)"**

| 维度 | 现有 (v1.0) | 目标 (v2.0) |
|------|------------|------------|
| 视觉风格 | Neo-Brutalism (高对比、粗边框) | Subtle Neo-Matte (柔和光影、哑光质感) |
| 色彩系统 | 黄橙渐变 + 黑色粗边框 | Zinc 中性色 + Teal 品牌色 |
| 交互模式 | 单次生成 | 批量生成 + 多比例并行 |
| 布局结构 | 双栏并列 | 浮动命令中心 + 时间线 Feed |

### 1.2 参考项目

- **Giga Peach**: 批量生成逻辑、任务队列架构、浮动命令中心 UI
- **CardForge Design Spec**: 微拟物视觉语言、组件规范、Tailwind 配置

---

## 2. 色彩系统

### 2.1 基础色调 (Neutral Palette)

使用 **Zinc (锌灰)** 色系，温润现代。

| 用途 | Tailwind 类 | 色值 |
|------|------------|------|
| 背景层 (Canvas) | `bg-zinc-50` | #FAFAFA |
| 表面层 (Surface) | `bg-white` | #FFFFFF |
| 凹陷层 (Recessed) | `bg-zinc-100` | #F4F4F5 |
| 次级文字 | `text-zinc-500` | #71717A |
| 主文字 | `text-zinc-700` | #3F3F46 |
| 标题文字 | `text-zinc-900` | #18181B |

### 2.2 品牌色 (Brand Color)

| 语义类名 | Tailwind 原生类 | 色值 | 用途 |
|----------|----------------|------|------|
| `bg-brand` | `bg-teal-700` | #0F766E | 主按钮、CTA |
| `bg-brand-light` | `bg-teal-50` | #F0FDFA | 选中态背景 |
| `bg-brand-dark` | `bg-teal-800` | #115E59 | 按钮 Hover |

### 2.3 语义色

| 状态 | 背景 | 边框 | 文字 |
|------|------|------|------|
| 成功 | `bg-teal-50` | `border-teal-500` | `text-teal-700` |
| 警告 | `bg-amber-50` | `border-amber-400` | `text-amber-700` |
| 错误 | `bg-red-50` | `border-red-500` | `text-red-700` |
| 加载中 | `bg-zinc-100` | `border-zinc-300` | `text-zinc-500` |

---

## 3. 光影与深度系统

### 3.1 凸起效果 (Elevation)

用于卡片、悬浮面板、按钮。

```css
shadow-neo-lift: 0 4px 20px -4px rgba(0,0,0,0.05), 0 -2px 10px -2px rgba(255,255,255,0.8);
shadow-neo-lift-hover: 0 6px 24px -4px rgba(0,0,0,0.08), 0 -2px 12px -2px rgba(255,255,255,0.9);
```

### 3.2 凹陷效果 (Depression)

用于输入框、搜索栏、选中态。

```css
shadow-neo-inset: inset 0 2px 4px rgba(0,0,0,0.04);
```

### 3.3 圆角系统

| 语义类名 | 像素值 | 用途 |
|----------|--------|------|
| `rounded-neo` | 12px | 按钮、输入框、小卡片 |
| `rounded-neo-lg` | 16px | 大卡片、模态框、面板 |

---

## 4. 布局架构

### 4.1 整体结构 (参考 Giga Peach)

```
┌─────────────────────────────────────────────────────────────┐
│ Header (Fixed Top)                                          │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ Logo  │ [Create] [Gallery] [Favorites] │ API Status    │ │
│ └─────────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Main Content Area (Scrollable)                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Create Tab: Generation Timeline (Bottom → Top)      │    │
│  │ Gallery Tab: Masonry Grid                           │    │
│  │ Favorites Tab: Filtered Masonry Grid                │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│ Floating Command Center (Fixed Bottom)                      │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ [Settings Tray - Collapsible]                           │ │
│ │ [Style Chips Row]                                       │ │
│ │ [Prompt Input] + [Reference Images] + [Generate Button] │ │
│ └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 响应式断点

| 断点 | 像素值 | 布局行为 |
|------|--------|----------|
| `sm` | 640px | 小平板，单列 |
| `md` | 768px | 平板横屏，侧边栏可见 |
| `lg` | 1024px | 桌面端，多列网格 |
| `xl` | 1280px | 大屏优化，4 列网格 |

---

## 5. 核心功能重制

### 5.1 从 Giga Peach 引入的功能

| 功能 | 描述 | 实现优先级 |
|------|------|-----------|
| **批量生成** | 单次生成 1-8 张图片 | P0 |
| **多比例并行** | 同时选择多个宽高比 (如 16:9 + 9:16) | P0 |
| **任务队列** | `GenerationTask` 对象追踪每个请求的生命周期 | P0 |
| **时间线 Feed** | 按批次分组展示历史生成，支持无限滚动 | P0 |
| **浮动命令中心** | 固定底部的输入区域，最大化图片浏览空间 | P0 |
| **风格预设系统** | 保存/编辑/删除风格，包含提示词前缀 + 参考图 | P1 |
| **Lightbox 浏览** | 全屏查看 + 键盘导航 (← → Esc) | P1 |
| **收藏夹** | 标记/取消收藏，独立 Tab 展示 | P1 |
| **IndexedDB 存储** | 本地持久化生成历史 (base64 图片) | P1 |
| **迭代工作流** | "Reuse" 按钮回填提示词 + 参考图到输入区 | P2 |

### 5.2 保留的 Nano Banana 功能

| 功能 | 描述 |
|------|------|
| 多 API 提供商 | 支持配置多个端点 + API Key |
| 模型列表获取 | 从端点自动拉取可用模型 |
| 双模式切换 | 图生图 / 文生图 |
| Gemini 专属配置 | 图片尺寸、宽高比、Google 搜索开关 |
| 提示词仓库 | 从远程仓库获取预设提示词 |
| SSE 流式解析 | 支持流式响应解码 |

---

## 6. 组件规范

### 6.1 Header

```html
<header class="fixed top-0 left-0 right-0 z-40 h-16 
               bg-white/95 backdrop-blur-sm shadow-neo-lift
               flex items-center justify-between px-6">
  <!-- Logo -->
  <div class="flex items-center gap-3 cursor-pointer group">
    <span class="text-3xl">🍌</span>
    <div>
      <h1 class="text-xl font-bold text-zinc-900">Nano Banana</h1>
      <p class="text-xs text-zinc-400">AI Image Generation Suite</p>
    </div>
  </div>
  
  <!-- Navigation Tabs -->
  <div class="flex bg-zinc-100 rounded-neo p-1 gap-1">
    <button class="px-4 py-2 rounded-lg text-sm font-medium 
                   transition-all
                   [&.active]:bg-white [&.active]:shadow-sm [&.active]:text-brand">
      Create
    </button>
    <!-- Gallery, Favorites... -->
  </div>
  
  <!-- API Status -->
  <button class="flex items-center gap-2 px-4 py-2 rounded-full text-xs font-medium 
                 border transition-all
                 [&.connected]:bg-teal-50 [&.connected]:border-teal-200 [&.connected]:text-teal-700
                 [&.disconnected]:bg-red-50 [&.disconnected]:border-red-200 [&.disconnected]:text-red-600">
    <span class="w-2 h-2 rounded-full bg-current"></span>
    <span>Connected</span>
  </button>
</header>
```

### 6.2 浮动命令中心

```html
<div class="fixed bottom-0 left-0 right-0 z-30 pb-6 px-4 pointer-events-none">
  <div class="max-w-4xl mx-auto pointer-events-auto space-y-3">
    
    <!-- Settings Tray (Collapsible) -->
    <div class="bg-white rounded-neo-lg shadow-neo-lift p-6 
                transition-all duration-300 origin-bottom
                [&.collapsed]:max-h-0 [&.collapsed]:opacity-0 [&.collapsed]:scale-95">
      <!-- Aspect Ratio Grid -->
      <!-- Resolution Selector -->
      <!-- Count Slider -->
    </div>
    
    <!-- Style Chips -->
    <div class="flex gap-2 overflow-x-auto pb-1 scrollbar-hide">
      <button class="flex items-center gap-2 px-4 py-2 rounded-full text-sm 
                     whitespace-nowrap border transition-all
                     [&.active]:bg-brand-light [&.active]:border-brand [&.active]:text-brand
                     bg-white border-zinc-200 text-zinc-600 hover:border-zinc-300">
        None
      </button>
      <!-- More style chips... -->
    </div>
    
    <!-- Main Input Area -->
    <div class="bg-white rounded-neo-lg shadow-neo-lift p-4">
      <div class="flex gap-4 items-end">
        <!-- Reference Images Thumbnails -->
        <div class="flex gap-2 flex-shrink-0">
          <!-- Image previews... -->
        </div>
        
        <!-- Prompt Textarea -->
        <textarea 
          class="flex-1 bg-zinc-100/80 shadow-neo-inset rounded-neo px-4 py-3 
                 resize-none outline-none min-h-[60px] max-h-[200px]
                 focus:bg-white focus:ring-2 focus:ring-teal-100 transition-all"
          placeholder="Describe your image..."></textarea>
        
        <!-- Action Buttons -->
        <div class="flex gap-2 flex-shrink-0">
          <button class="p-3 rounded-neo bg-zinc-100 text-zinc-500 
                         hover:bg-zinc-200 transition-colors">
            <!-- Settings Icon -->
          </button>
          <button class="px-6 py-3 rounded-neo bg-brand text-white font-semibold
                         shadow-md hover:bg-brand-dark transition-colors
                         disabled:opacity-50 disabled:cursor-not-allowed">
            Generate
          </button>
        </div>
      </div>
    </div>
  </div>
</div>
```

### 6.3 Generation Timeline (Create Tab)

```html
<div class="space-y-12 pb-48 pt-20">
  <!-- Batch Group -->
  <div class="group relative">
    <!-- Timeline Line -->
    <div class="absolute -left-8 top-6 bottom-0 w-px bg-gradient-to-b 
                from-brand/50 to-transparent hidden md:block"></div>
    
    <!-- Batch Header -->
    <div class="mb-6">
      <p class="text-zinc-800 text-lg font-medium leading-relaxed max-w-4xl">
        {{ prompt }}
      </p>
      
      <!-- Reference Images Preview -->
      <div class="flex gap-2 mt-3">
        <div class="w-12 h-12 rounded-neo overflow-hidden border border-zinc-200">
          <img src="..." class="w-full h-full object-cover" />
        </div>
      </div>
      
      <!-- Meta Info -->
      <div class="flex items-center gap-3 mt-2 text-sm text-zinc-400">
        <span>2026/01/15 09:30</span>
        <span>•</span>
        <span class="text-brand font-medium">Cinematic Style</span>
        <span>•</span>
        <span>2K</span>
      </div>
    </div>
    
    <!-- Images Grid by Ratio -->
    <div class="space-y-6">
      <div>
        <div class="text-xs uppercase font-bold text-zinc-400 mb-3">16:9</div>
        <div class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
          <!-- ImageCard components -->
        </div>
      </div>
    </div>
    
    <!-- Batch Actions (Hover) -->
    <div class="mt-4 flex justify-end gap-3 opacity-0 group-hover:opacity-100 transition-opacity">
      <button class="flex items-center gap-2 text-sm text-zinc-500 hover:text-zinc-700 
                     bg-white shadow-neo-lift px-4 py-2 rounded-full">
        Reuse
      </button>
      <button class="flex items-center gap-2 text-sm text-brand 
                     bg-brand-light px-4 py-2 rounded-full">
        Regenerate Batch
      </button>
    </div>
  </div>
</div>
```

### 6.4 ImageCard

```html
<div class="relative group rounded-neo-lg overflow-hidden bg-zinc-100 
            shadow-neo-lift hover:shadow-neo-lift-hover transition-all cursor-pointer"
     :class="aspectClasses[aspectRatio]">
  
  <!-- Loading State -->
  <div v-if="status === 'generating'" class="absolute inset-0 flex items-center justify-center">
    <div class="w-8 h-8 border-2 border-brand/30 border-t-brand rounded-full animate-spin"></div>
  </div>
  
  <!-- Error State -->
  <div v-else-if="status === 'error'" class="absolute inset-0 flex items-center justify-center bg-red-50">
    <span class="text-sm text-red-500">Failed</span>
  </div>
  
  <!-- Success State -->
  <template v-else>
    <img :src="imageUrl" class="w-full h-full object-cover 
                                 transition-transform duration-500 group-hover:scale-105" />
    
    <!-- Top Bar (Ratio + Favorite) -->
    <div class="absolute top-0 left-0 right-0 p-2 flex justify-between items-start 
                opacity-0 group-hover:opacity-100 transition-opacity">
      <span class="px-2 py-1 bg-black/50 backdrop-blur rounded text-[10px] font-mono text-white">
        {{ aspectRatio }}
      </span>
      <button class="p-1.5 rounded-full backdrop-blur transition-colors
                     [&.favorited]:bg-red-500/20 [&.favorited]:text-red-500
                     bg-black/40 text-white hover:bg-white/20">
        <!-- Heart Icon -->
      </button>
    </div>
    
    <!-- Bottom Overlay Actions -->
    <div class="absolute inset-x-0 bottom-0 bg-gradient-to-t from-black/80 via-black/40 to-transparent 
                p-3 pt-8 opacity-0 group-hover:opacity-100 transition-opacity">
      <div class="flex justify-between items-center">
        <div class="flex gap-2">
          <button class="p-2 bg-white/20 backdrop-blur rounded-full hover:bg-brand text-white transition-colors">
            <!-- Iterate Icon -->
          </button>
          <a class="p-2 bg-white/20 backdrop-blur rounded-full hover:bg-teal-500 text-white transition-colors">
            <!-- Download Icon -->
          </a>
        </div>
        <button class="p-2 bg-white/20 backdrop-blur rounded-full hover:bg-red-500 text-white transition-colors">
          <!-- Delete Icon -->
        </button>
      </div>
    </div>
  </template>
</div>
```

### 6.5 Settings Tray

```html
<div class="grid grid-cols-1 md:grid-cols-2 gap-8">
  
  <!-- Aspect Ratios -->
  <div class="space-y-3">
    <label class="text-xs uppercase font-bold text-zinc-500 tracking-wider">
      Aspect Ratio
    </label>
    <div class="grid grid-cols-5 gap-2">
      <button v-for="ratio in ratios" 
              class="flex flex-col items-center justify-center gap-1.5 p-2 rounded-neo h-16
                     border transition-all cursor-pointer
                     [&.active]:bg-brand-light [&.active]:border-brand [&.active]:text-brand
                     bg-zinc-50 border-zinc-200 text-zinc-500 hover:bg-zinc-100">
        <!-- Ratio Visual Box -->
        <div class="border-2 rounded-sm transition-all" 
             :style="{ width: ratioWidth, height: ratioHeight }"></div>
        <span class="text-[10px] font-mono">{{ ratio }}</span>
      </button>
    </div>
  </div>
  
  <!-- Resolution + Count -->
  <div class="space-y-6">
    <!-- Resolution -->
    <div class="space-y-3">
      <label class="text-xs uppercase font-bold text-zinc-500 tracking-wider">
        Resolution
      </label>
      <div class="flex bg-zinc-100 rounded-neo p-1 gap-1">
        <button v-for="res in ['1K', '2K', '4K']"
                class="flex-1 py-2 text-sm rounded-lg transition-all
                       [&.active]:bg-white [&.active]:shadow-sm [&.active]:text-zinc-800
                       text-zinc-500 hover:text-zinc-700">
          {{ res }}
        </button>
      </div>
    </div>
    
    <!-- Image Count -->
    <div class="space-y-3">
      <div class="flex justify-between">
        <label class="text-xs uppercase font-bold text-zinc-500 tracking-wider">
          Image Count
        </label>
        <span class="text-sm font-mono text-brand">{{ count }}</span>
      </div>
      <input type="range" min="1" max="8" v-model="count"
             class="w-full h-2 bg-zinc-200 rounded-lg appearance-none cursor-pointer
                    accent-brand" />
    </div>
  </div>
</div>
```

### 6.6 Style Preset Modal

```html
<div class="fixed inset-0 z-50 flex items-center justify-center">
  <!-- Backdrop -->
  <div class="absolute inset-0 bg-zinc-900/50 backdrop-blur-sm"></div>
  
  <!-- Modal -->
  <div class="relative bg-white rounded-neo-lg shadow-2xl p-6 max-w-lg w-full mx-4">
    <!-- Header -->
    <div class="flex items-center justify-between mb-6">
      <h2 class="text-xl font-bold text-zinc-900">
        {{ isEditing ? 'Edit Style' : 'Create Style' }}
      </h2>
      <button class="p-2 text-zinc-400 hover:text-zinc-600 rounded-full hover:bg-zinc-100">
        <!-- X Icon -->
      </button>
    </div>
    
    <!-- Form -->
    <div class="space-y-4">
      <div class="space-y-1">
        <label class="text-sm font-medium text-zinc-700">Name</label>
        <input type="text" v-model="name"
               class="w-full bg-zinc-100/80 shadow-neo-inset rounded-neo px-4 py-2.5
                      outline-none focus:bg-white focus:ring-2 focus:ring-teal-100 transition-all" />
      </div>
      
      <div class="space-y-1">
        <label class="text-sm font-medium text-zinc-700">Description / Prompt Prefix</label>
        <textarea v-model="description" rows="4"
                  class="w-full bg-zinc-100/80 shadow-neo-inset rounded-neo px-4 py-2.5
                         resize-none outline-none focus:bg-white focus:ring-2 focus:ring-teal-100 transition-all">
        </textarea>
      </div>
      
      <div class="space-y-1">
        <label class="text-sm font-medium text-zinc-700">Reference Images (Optional)</label>
        <div class="flex flex-wrap gap-2 bg-zinc-50 rounded-neo p-3 min-h-[80px]">
          <div v-for="img in referenceImages" 
               class="relative w-16 h-16 rounded-neo overflow-hidden border border-zinc-200 group">
            <img :src="img" class="w-full h-full object-cover" />
            <button class="absolute top-0.5 right-0.5 w-5 h-5 bg-red-500 text-white 
                           rounded-full text-xs opacity-0 group-hover:opacity-100 transition-opacity">
              ×
            </button>
          </div>
          <button class="w-16 h-16 border-2 border-dashed border-zinc-300 rounded-neo
                         flex items-center justify-center text-zinc-400 hover:border-brand hover:text-brand
                         transition-colors">
            +
          </button>
        </div>
      </div>
    </div>
    
    <!-- Actions -->
    <div class="flex justify-between mt-6">
      <button v-if="isEditing" 
              class="text-sm text-red-500 hover:text-red-700 px-4 py-2">
        Delete
      </button>
      <div class="flex gap-3 ml-auto">
        <button class="bg-white text-zinc-700 shadow-neo-lift px-4 py-2 rounded-neo
                       hover:bg-zinc-50 transition-colors">
          Cancel
        </button>
        <button class="bg-brand text-white px-6 py-2 rounded-neo shadow-md
                       hover:bg-brand-dark transition-colors">
          Save
        </button>
      </div>
    </div>
  </div>
</div>
```

### 6.7 Lightbox

```html
<div class="fixed inset-0 z-50 bg-zinc-900/95 backdrop-blur-sm 
            flex items-center justify-center">
  
  <!-- Navigation -->
  <button class="absolute left-4 top-1/2 -translate-y-1/2 p-3 
                 bg-white/10 hover:bg-white/20 rounded-full text-white transition-colors">
    <!-- ChevronLeft Icon -->
  </button>
  <button class="absolute right-4 top-1/2 -translate-y-1/2 p-3 
                 bg-white/10 hover:bg-white/20 rounded-full text-white transition-colors">
    <!-- ChevronRight Icon -->
  </button>
  
  <!-- Close -->
  <button class="absolute top-4 right-4 p-3 
                 bg-white/10 hover:bg-white/20 rounded-full text-white transition-colors">
    <!-- X Icon -->
  </button>
  
  <!-- Image -->
  <img :src="currentImage.url" 
       class="max-w-[90vw] max-h-[85vh] object-contain rounded-neo-lg shadow-2xl" />
  
  <!-- Bottom Bar -->
  <div class="absolute bottom-4 left-1/2 -translate-x-1/2 
              flex items-center gap-4 bg-black/50 backdrop-blur rounded-full px-6 py-3">
    <button class="p-2 text-white/80 hover:text-white transition-colors">
      <!-- Heart Icon -->
    </button>
    <button class="p-2 text-white/80 hover:text-white transition-colors">
      <!-- Download Icon -->
    </button>
    <button class="p-2 text-white/80 hover:text-white transition-colors">
      <!-- Iterate Icon -->
    </button>
    <span class="text-white/60 text-sm font-mono">
      {{ currentIndex + 1 }} / {{ images.length }}
    </span>
  </div>
</div>
```

### 6.8 Toast 通知

```html
<div class="fixed bottom-6 right-6 z-[60] space-y-3">
  <div v-for="toast in toasts" 
       class="bg-white rounded-neo px-4 py-3 shadow-neo-lift 
              flex items-center gap-3 min-w-[280px]
              border-l-4 transition-all duration-300"
       :class="{
         'border-teal-500': toast.type === 'success',
         'border-red-500': toast.type === 'error',
         'border-zinc-400': toast.type === 'loading'
       }">
    <!-- Icon -->
    <span v-if="toast.type === 'success'" class="text-teal-500">✓</span>
    <span v-else-if="toast.type === 'error'" class="text-red-500">✕</span>
    <div v-else class="w-4 h-4 border-2 border-zinc-300 border-t-zinc-600 rounded-full animate-spin"></div>
    
    <!-- Message -->
    <span class="text-zinc-700 text-sm">{{ toast.message }}</span>
  </div>
</div>
```

---

## 7. 状态管理

### 7.1 全局状态结构

```typescript
interface AppState {
  // API Configuration
  apiConfigs: ApiProviderConfig[]
  activeProviderId: string
  
  // Generation Parameters
  params: {
    aspectRatios: AspectRatio[]  // 多选
    resolution: '1K' | '2K' | '4K'
    count: number  // 1-8
  }
  
  // Inputs
  prompt: string
  referenceImages: string[]  // base64[]
  selectedStyleId: string | null
  
  // Generation State
  tasks: GenerationTask[]
  
  // Gallery
  gallery: GeneratedImage[]
  galleryHasMore: boolean
  
  // UI State
  activeTab: 'create' | 'gallery' | 'favorites'
  showSettings: boolean
  lightbox: {
    isOpen: boolean
    images: GeneratedImage[]
    currentIndex: number
  }
  
  // Presets
  stylePresets: StylePreset[]
}
```

### 7.2 数据类型定义

```typescript
interface ApiProviderConfig {
  id: string
  name: string
  apiKey: string
  endpoint: string
  model: string
}

interface StylePreset {
  id: string
  name: string
  description: string  // Prompt prefix
  icon?: string
  referenceImages?: string[]
}

interface GenerationTask {
  id: string
  batchId: string
  status: 'pending' | 'generating' | 'success' | 'error'
  aspectRatio: AspectRatio
  prompt: string
  referenceImages?: string[]
  data?: GeneratedImage
  error?: string
}

interface GeneratedImage {
  id: string
  batchId: string
  url: string  // base64 data URI
  prompt: string
  aspectRatio: AspectRatio
  resolution: string
  timestamp: number
  styleId?: string
  referenceImages?: string[]
  isFavorite?: boolean
}

type AspectRatio = '1:1' | '2:3' | '3:2' | '3:4' | '4:3' | '4:5' | '5:4' | '9:16' | '16:9' | '21:9'
```

### 7.3 持久化策略

| 数据 | 存储方式 | Key 前缀 |
|------|----------|----------|
| API 配置 | LocalStorage | `nano-banana-api-configs` |
| 活跃提供商 ID | LocalStorage | `nano-banana-active-provider` |
| 生成参数 | LocalStorage | `nano-banana-params` |
| 风格预设 | IndexedDB | `presets` store |
| 生成历史 | IndexedDB | `images` store |
| 模型缓存 | LocalStorage | `nano-banana-models-{endpoint}` |

---

## 8. Tailwind 配置

```javascript
// tailwind.config.js
export default {
  content: ['./index.html', './src/**/*.{vue,js,ts}'],
  theme: {
    extend: {
      colors: {
        brand: {
          DEFAULT: '#0f766e',
          light: '#f0fdfa',
          dark: '#115e59',
        },
      },
      borderRadius: {
        'neo': '12px',
        'neo-lg': '16px',
      },
      boxShadow: {
        'neo-lift': '0 4px 20px -4px rgba(0,0,0,0.05), 0 -2px 10px -2px rgba(255,255,255,0.8)',
        'neo-lift-hover': '0 6px 24px -4px rgba(0,0,0,0.08), 0 -2px 12px -2px rgba(255,255,255,0.9)',
        'neo-inset': 'inset 0 2px 4px rgba(0,0,0,0.04)',
      },
      animation: {
        'fade-in': 'fade-in 0.3s ease-out',
        'slide-up': 'slide-up 0.3s ease-out',
      },
      keyframes: {
        'fade-in': {
          '0%': { opacity: '0' },
          '100%': { opacity: '1' },
        },
        'slide-up': {
          '0%': { opacity: '0', transform: 'translateY(10px)' },
          '100%': { opacity: '1', transform: 'translateY(0)' },
        },
      },
    },
  },
}
```

---

## 9. 组件清单

### 9.1 核心组件

| 组件 | 文件路径 | 职责 |
|------|----------|------|
| `App.vue` | `src/App.vue` | 主协调器，全局状态管理 |
| `AppHeader.vue` | `src/components/AppHeader.vue` | 顶部导航栏 |
| `CommandCenter.vue` | `src/components/CommandCenter.vue` | 浮动输入区域 |
| `SettingsTray.vue` | `src/components/SettingsTray.vue` | 可折叠设置面板 |
| `StyleChips.vue` | `src/components/StyleChips.vue` | 风格预设横向选择器 |
| `PromptInput.vue` | `src/components/PromptInput.vue` | 提示词输入框 |
| `ReferenceImages.vue` | `src/components/ReferenceImages.vue` | 参考图上传/预览 |

### 9.2 内容展示组件

| 组件 | 文件路径 | 职责 |
|------|----------|------|
| `GenerationTimeline.vue` | `src/components/GenerationTimeline.vue` | Create Tab 时间线 |
| `BatchGroup.vue` | `src/components/BatchGroup.vue` | 单个批次分组 |
| `ImageCard.vue` | `src/components/ImageCard.vue` | 单张图片卡片 |
| `GalleryGrid.vue` | `src/components/GalleryGrid.vue` | 瀑布流网格 |
| `Lightbox.vue` | `src/components/Lightbox.vue` | 全屏图片浏览 |

### 9.3 配置组件

| 组件 | 文件路径 | 职责 |
|------|----------|------|
| `ApiConfigModal.vue` | `src/components/ApiConfigModal.vue` | API 配置弹窗 |
| `StylePresetModal.vue` | `src/components/StylePresetModal.vue` | 风格预设编辑弹窗 |
| `AspectRatioSelector.vue` | `src/components/AspectRatioSelector.vue` | 宽高比多选器 |
| `ResolutionSelector.vue` | `src/components/ResolutionSelector.vue` | 分辨率选择器 |
| `CountSlider.vue` | `src/components/CountSlider.vue` | 生成数量滑块 |

### 9.4 通用组件

| 组件 | 文件路径 | 职责 |
|------|----------|------|
| `Modal.vue` | `src/components/ui/Modal.vue` | 通用模态框容器 |
| `Toast.vue` | `src/components/ui/Toast.vue` | 通知提示 |
| `Button.vue` | `src/components/ui/Button.vue` | 按钮 (主要/次要/危险) |
| `Input.vue` | `src/components/ui/Input.vue` | 输入框 |
| `Toggle.vue` | `src/components/ui/Toggle.vue` | 开关 |

---

## 10. 实施路线图

### Phase 1: 基础架构重制 (Week 1-2)

- [ ] 更新 Tailwind 配置 (新增自定义 token)
- [ ] 重构 `App.vue` 状态管理结构
- [ ] 实现新 Header 组件
- [ ] 实现浮动命令中心骨架
- [ ] 集成 IndexedDB 存储层

### Phase 2: 核心功能迁移 (Week 3-4)

- [ ] 实现批量生成任务队列
- [ ] 实现多比例选择器
- [ ] 迁移 API 服务层 (保留 SSE 支持)
- [ ] 实现 GenerationTimeline 组件
- [ ] 实现 ImageCard 组件

### Phase 3: 增强功能 (Week 5-6)

- [ ] 实现风格预设系统
- [ ] 实现 Lightbox 浏览
- [ ] 实现收藏夹功能
- [ ] 实现无限滚动加载
- [ ] 迭代工作流 (Reuse/Regenerate)

### Phase 4: 优化与打磨 (Week 7-8)

- [ ] 响应式布局优化
- [ ] 动画与过渡效果
- [ ] 性能优化 (虚拟滚动)
- [ ] 错误处理与边界情况
- [ ] 文档与测试

---

## 附录 A: 图标库

使用 **Lucide Icons** (与 Giga Peach 保持一致)，通过 `lucide-vue-next` 引入。

常用图标：
- `Sparkles` - 生成/AI
- `Settings` - 设置
- `History` - 历史
- `Heart` - 收藏
- `Download` - 下载
- `Trash2` - 删除
- `RefreshCw` - 重新生成
- `ChevronLeft/Right` - 导航
- `X` - 关闭
- `Plus` - 添加
- `Key` - API 密钥

---

## 附录 B: 交互动效规范

| 场景 | Duration | Easing | 效果 |
|------|----------|--------|------|
| 悬浮/聚焦 | 150ms | ease-out | 阴影加深、背景变化 |
| 展开/收起 | 200ms | ease-out | 高度/透明度变化 |
| 模态框 | 300ms | ease-out | 缩放 + 透明度 |
| 页面切换 | 200ms | ease-out | 淡入淡出 |
| 图片加载 | 500ms | ease-out | 缩放 hover 效果 |

---

## 修订历史

| 版本 | 日期 | 修改内容 |
|------|------|----------|
| 1.0 | 2026-01-15 | 初始版本，基于 Giga Peach 功能分析与 CardForge 设计规范融合 |
