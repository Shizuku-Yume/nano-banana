# Nano Banana 项目交接文档

**日期**: 2026-01-17
**状态**: 功能开发中

---

## 项目概述

Nano Banana 是一个纯前端的 AI 图像生成 Web UI，基于 Vue 3 + TypeScript + TailwindCSS，直接调用 OpenRouter API 或兼容端点（如 CliProxyAPI）进行图像生成。

## 项目位置

| 项目 | 路径 |
|------|------|
| Nano Banana | `/home/shizuku/project/nano-banana/` |

---

## 本次会话完成的工作

### 1. 分辨率和 Google 搜索参数修复

**问题**: UI 有分辨率选择 (1K/2K/4K)，但 API 请求中没有传递该参数

**修复**:
- `App.vue`: 添加 `enableGoogleSearch` 状态
- `App.vue`: `executeTask()` 现在传递 `imageSize` 和 `enableGoogleSearch` 参数
- `SettingsTray.vue`: 添加 Google 搜索开关 UI（仅 Gemini 3 Pro Image 支持）

### 2. 模型列表优化

**问题**: 模型列表无排序，生图模型没有标识

**修复**:
- 生图模型添加 🖼️ 标识
- 生图模型排序到列表前面
- 通过 `m.id.toLowerCase().includes('image')` 判断是否为生图模型

### 3. SSE 流解析修复

**问题**: 服务端返回图片数据后连接中断，导致 network error，已传输的图片数据丢失

**根因分析**:
- 成功响应格式: `data: {图片}\n\ndata: {finish}\n\ndata: [DONE]\n`
- 失败响应: 只有第一行 `data: {图片}`，没有后续行
- 原代码只处理以 `\n` 结尾的完整行，buffer 中的剩余数据被丢弃

**修复** (`src/services/api.ts`):
- 添加 try-catch 处理连接中断
- 流结束后处理 buffer 中的剩余数据
- 即使连接中断，只要有有效数据就返回

### 4. 并发数调整

- `MAX_CONCURRENT` 从 3 改为 4

---

## 提交记录

```
06cc932 chore: increase MAX_CONCURRENT from 3 to 4
b81a6d2 fix: process remaining buffer data when SSE stream is interrupted
19ce827 fix: handle SSE stream interruption gracefully
d095434 feat: add resolution/Google Search params to API and improve model list
```

---

## 已知问题

### 1. 请求串行执行

**现象**: 设置 `MAX_CONCURRENT = 4`，但 Network 面板显示请求是串行完成的

**原因**: 不是前端代码问题，可能是：
- `cli.shizukuyume.fun` 服务端队列限制
- Google Gemini API 对并发请求的限制

**验证方法**: 检查服务端日志或配置

### 2. 间歇性连接中断

**现象**: 部分请求在数据传输完成后连接被关闭

**原因**: 可能是服务端与 Google API 之间的连接问题（B 段），不是用户网络问题（A 段）

**证据**: 失败请求传输了 6-11MB 数据，说明用户网络正常

---

## 开发命令

```bash
cd /home/shizuku/project/nano-banana
npm install          # 安装依赖
npm run dev          # 开发服务器 @ localhost:3000
npm run build        # 生产构建 → dist/
npx vue-tsc --noEmit # 类型检查（有预存在的类型错误，不影响构建）
```

---

## 关键文件

| 文件 | 说明 |
|------|------|
| `src/App.vue` | 主应用，状态管理，生成逻辑 |
| `src/services/api.ts` | API 调用，SSE 解析，重试逻辑 |
| `src/components/SettingsTray.vue` | 分辨率、宽高比、Google 搜索设置 |
| `src/components/ApiConfigModal.vue` | API 配置和模型选择 |
| `src/types/index.ts` | TypeScript 类型定义 |
| `AGENTS.md` | 项目知识库（给 AI 看的） |

---

## API 参数参考 (Gemini 3 Pro Image)

| 参数 | 值 | 说明 |
|------|-----|------|
| `image_config.aspect_ratio` | `1:1`, `16:9`, 等 | 宽高比 |
| `image_config.image_size` | `1K`, `2K`, `4K` | 分辨率 |
| `tools[].google_search` | `{}` | 启用 Google 搜索（实时数据） |
| `modalities` | `["image", "text"]` | 请求图片输出 |
