---
title: SvgIcon 组件集成指南
description: 在任意 vite 项目中集成 soybean-admin 的 SvgIcon 组件
date: 2025-11-28T16:00:00.000+00:00
lang: zh
duration: 10min
---

本文档详细说明了如何将 [soybean-admin](https://github.com/soybeanjs/soybean-admin) 的 `SvgIcon` 组件集成到其他项目中。该组件支持 Iconify 在线/离线图标以及本地 SVG 图标。

## 1. 核心功能

- **Iconify 支持**: 集成 `@iconify/vue`，支持海量图标库。
- **本地 SVG 支持**: 集成 `vite-plugin-svg-icons`，支持加载本地 SVG 文件。
- **智能回退**: 优先渲染本地图标，未找到则尝试渲染 Iconify 图标。

## 2. 依赖安装

在目标项目中安装必要的依赖：

```bash
# 运行时依赖
pnpm add @iconify/vue

# 开发依赖 (用于构建 SVG 雪碧图)
pnpm add -D vite-plugin-svg-icons
```

## 3. 组件源码

创建文件 `src/components/custom/svg-icon.vue`：

```vue
<script setup lang="ts">
import { Icon } from '@iconify/vue'
import { computed, useAttrs } from 'vue'

defineOptions({ name: 'SvgIcon', inheritAttrs: false })

const props = defineProps<Props>()

/**
 * Props
 *
 * - 支持 iconify 和 本地 svg icon
 * - 如果同时传递了 icon 和 localIcon，localIcon 会优先渲染
 */
interface Props {
  /** Iconify icon 名称 */
  icon?: string
  /** 本地 svg icon 名称 */
  localIcon?: string
}

const attrs = useAttrs()

const bindAttrs = computed<{ class: string, style: string }>(() => ({
  class: (attrs.class as string) || '',
  style: (attrs.style as string) || ''
}))

const symbolId = computed(() => {
  // 依赖环境变量 VITE_ICON_LOCAL_PREFIX
  const { VITE_ICON_LOCAL_PREFIX: prefix } = import.meta.env

  const defaultLocalIcon = 'no-icon'

  const icon = props.localIcon || defaultLocalIcon

  return `#${prefix}-${icon}`
})

/** 如果传递了 localIcon，优先渲染 localIcon */
const renderLocalIcon = computed(() => props.localIcon || !props.icon)
</script>

<template>
  <template v-if="renderLocalIcon">
    <svg aria-hidden="true" width="1em" height="1em" v-bind="bindAttrs">
      <use :xlink:href="symbolId" fill="currentColor" />
    </svg>
  </template>
  <template v-else>
    <Icon v-if="icon" :icon="icon" v-bind="bindAttrs" />
  </template>
</template>
```

## 4. Vite 配置 (`vite.config.ts`)

配置 `vite-plugin-svg-icons` 以处理本地 SVG 文件：

```typescript
import path from 'node:path'
import process from 'node:process'
import { createSvgIconsPlugin } from 'vite-plugin-svg-icons'

export default defineConfig({
  plugins: [
    // ...
    createSvgIconsPlugin({
      // 指定本地 SVG 图标存放目录
      iconDirs: [path.join(process.cwd(), 'src/assets/svg-icon')],
      // 指定 symbolId 格式 (必须与组件内的 symbolId 计算逻辑匹配)
      // 推荐使用环境变量配置前缀
      symbolId: `${process.env.VITE_ICON_LOCAL_PREFIX || 'icon-local'}-[dir]-[name]`,
      inject: 'body-last',
      customDomId: '__SVG_ICON_LOCAL__'
    })
  ]
})
```

## 5. 环境变量 (`.env`)

配置图标前缀变量：

```properties
# 本地 SVG 图标前缀 (必须与 vite 配置中的 symbolId 前缀一致)
VITE_ICON_LOCAL_PREFIX=icon-local
```

### 为什么需要 `VITE_ICON_LOCAL_PREFIX`？

`VITE_ICON_LOCAL_PREFIX` 起到了 **命名空间** 的作用，主要解决了两个问题：

1.  **避免 ID 冲突**: 页面中可能存在多个 SVG `symbol`，添加前缀可以确保我们生成的本地图标 ID 是唯一的（例如 `icon-local-logo`），避免与其他库或图标冲突。
2.  **建立连接**:
    - **构建时**: `vite-plugin-svg-icons` 使用该前缀生成 SVG symbol 的 ID（如 `<symbol id="icon-local-logo">`）。
    - **运行时**: `svg-icon.vue` 组件使用该前缀拼接出 `<use xlink:href="#icon-local-logo">` 的引用地址。
    - **必须一致**: 因此，`.env` 中的配置必须与 `vite.config.ts` 中的配置保持一致，否则组件将无法找到对应的图标。

## 6. 全局注册

在入口文件 `src/main.ts` 中引入注册脚本：

```typescript
import 'virtual:svg-icons-register'
```

## 7. 使用方法

### 7.1 在模板中使用 (Template)

```vue
<template>
  <!-- 1. 使用 Iconify 图标 -->
  <SvgIcon icon="mdi:home" class="text-24px" />

  <!-- 2. 使用本地 SVG 图标 -->
  <!-- 假设存在 src/assets/svg-icon/logo.svg -->
  <SvgIcon local-icon="logo" class="text-32px text-primary" />
</template>
```

### 7.2 在渲染函数中使用 (Render Function)

在某些场景下（如 Naive UI 的菜单配置 `renderIcon`，或 JSX/TSX 开发），你需要使用 Vue 的 `h` 函数来渲染组件。

```typescript
import { h } from 'vue'
import SvgIcon from '@/components/custom/svg-icon.vue'

/**
 * 渲染 Iconify 图标
 * @param icon - Iconify 图标名称 (如 'mdi:home')
 */
function renderIcon(icon: string) {
  return () => h(SvgIcon, { icon })
}

/**
 * 渲染本地 SVG 图标
 * @param localIcon - 本地 SVG 文件名 (如 'logo')
 */
function renderLocalIcon(localIcon: string) {
  return () => h(SvgIcon, { localIcon })
}

// 示例：在 Naive UI 菜单配置中使用
const menuOptions = [
  {
    label: '首页',
    key: 'home',
    icon: renderIcon('mdi:home')
  },
  {
    label: '关于',
    key: 'about',
    icon: renderLocalIcon('about-us') // 对应 src/assets/svg-icon/about-us.svg
  }
]
```
