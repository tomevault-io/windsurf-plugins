---
trigger: always_on
description: CSS 和样式规范（Tailwind 4 + Less）
---


# CSS 和样式规范

本项目使用 Tailwind 4 + Less + UnoCSS

## 样式优先级

### 1. Tailwind 工具类（最优先）
```vue
<template>
  <!-- ✅ 优先使用 Tailwind 类 -->
  <div class="flex items-center justify-between gap-4 p-4">
    <div class="w-full max-w-md">
      <p class="text-lg font-semibold text-gray-900">标题</p>
      <p class="text-sm text-gray-500">描述文本</p>
    </div>
  </div>
</template>
```

### 2. Naive UI 组件（次优先）
```vue
<template>
  <!-- ✅ 使用 Naive UI 组件的内置样式 -->
  <n-button type="primary" size="large" round>按钮</n-button>
  <n-card title="卡片标题" :bordered="false" size="small">
    卡片内容
  </n-card>
</template>
```

### 3. Less 样式（仅在必要时使用）
```vue
<style lang="less" scoped>
// ⚠️ 仅用于动态样式、复杂动画或 Tailwind 无法实现的样式
.custom-gradient {
  background: linear-gradient(135deg, @primary-color, @secondary-color);
}
</style>
```

## Tailwind 使用规范

### 布局
```vue
<template>
  <!-- ✅ Flexbox 布局 -->
  <div class="flex flex-col gap-4">
    <div class="flex items-center justify-between">
      <span>标题</span>
      <button>操作</button>
    </div>
  </div>
  
  <!-- ✅ Grid 布局 -->
  <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
    <div>项目 1</div>
    <div>项目 2</div>
    <div>项目 3</div>
  </div>
  
  <!-- ✅ 容器 -->
  <div class="container mx-auto px-4">
    内容
  </div>
</template>
```

### 间距
```vue
<template>
  <!-- ✅ Padding 和 Margin -->
  <div class="p-4">内边距 1rem</div>
  <div class="px-6 py-3">水平 1.5rem，垂直 0.75rem</div>
  <div class="mt-4 mb-6">上边距 1rem，下边距 1.5rem</div>
  
  <!-- ✅ Gap（用于 flex 和 grid）-->
  <div class="flex gap-2">间距 0.5rem</div>
  <div class="flex gap-4">间距 1rem</div>
  <div class="flex gap-x-4 gap-y-2">水平 1rem，垂直 0.5rem</div>
</template>
```

### 尺寸
```vue
<template>
  <!-- ✅ 宽度 -->
  <div class="w-full">100% 宽度</div>
  <div class="w-1/2">50% 宽度</div>
  <div class="w-64">16rem (256px) 宽度</div>
  <div class="min-w-0 max-w-md">最小宽度 0，最大宽度 28rem</div>
  
  <!-- ✅ 高度 -->
  <div class="h-screen">100vh 高度</div>
  <div class="h-64">16rem 高度</div>
  <div class="min-h-0 max-h-96">最小高度 0，最大高度 24rem</div>
</template>
```

### 颜色
```vue
<template>
  <!-- ✅ 文字颜色 -->
  <p class="text-gray-900">深色文字</p>
  <p class="text-gray-500">灰色文字</p>
  <p class="text-blue-600">蓝色文字</p>
  
  <!-- ✅ 背景颜色 -->
  <div class="bg-white">白色背景</div>
  <div class="bg-gray-100">浅灰背景</div>
  <div class="bg-blue-500">蓝色背景</div>
  
  <!-- ✅ 边框颜色 -->
  <div class="border border-gray-300">灰色边框</div>
  <div class="border-2 border-blue-500">蓝色边框</div>
</template>
```

### 文字样式
```vue
<template>
  <!-- ✅ 字体大小 -->
  <p class="text-xs">0.75rem (12px)</p>
  <p class="text-sm">0.875rem (14px)</p>
  <p class="text-base">1rem (16px)</p>
  <p class="text-lg">1.125rem (18px)</p>
  <p class="text-xl">1.25rem (20px)</p>
  <p class="text-2xl">1.5rem (24px)</p>
  
  <!-- ✅ 字体粗细 -->
  <p class="font-normal">普通</p>
  <p class="font-medium">中等</p>
  <p class="font-semibold">半粗</p>
  <p class="font-bold">粗体</p>
  
  <!-- ✅ 文字对齐 -->
  <p class="text-left">左对齐</p>
  <p class="text-center">居中</p>
  <p class="text-right">右对齐</p>
  
  <!-- ✅ 文字溢出 -->
  <p class="truncate">单行截断</p>
  <p class="line-clamp-2">两行截断</p>
  <p class="overflow-hidden text-ellipsis">溢出隐藏</p>
</template>
```

### 边框和圆角
```vue
<template>
  <!-- ✅ 圆角 -->
  <div class="rounded">0.25rem 圆角</div>
  <div class="rounded-md">0.375rem 圆角</div>
  <div class="rounded-lg">0.5rem 圆角</div>
  <div class="rounded-full">完全圆形</div>
  
  <!-- ✅ 边框 -->
  <div class="border">1px 边框</div>
  <div class="border-2">2px 边框</div>
  <div class="border-t">顶部边框</div>
  <div class="border-b">底部边框</div>
</template>
```

### 阴影
```vue
<template>
  <!-- ✅ 阴影效果 -->
  <div class="shadow-sm">小阴影</div>
  <div class="shadow">中等阴影</div>
  <div class="shadow-md">中大阴影</div>
  <div class="shadow-lg">大阴影</div>
  <div class="shadow-xl">超大阴影</div>
  
  <!-- ✅ 无阴影 -->
  <div class="shadow-none">无阴影</div>
</template>
```

### 响应式设计
```vue
<template>
  <!-- ✅ 断点：sm(640px) md(768px) lg(1024px) xl(1280px) 2xl(1536px) -->
  <div class="w-full md:w-1/2 lg:w-1/3">
    <!-- 移动端全宽，平板半宽，桌面三分之一宽 -->
  </div>
  
  <div class="text-sm md:text-base lg:text-lg">
    <!-- 响应式字体大小 -->
  </div>
  
  <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4">
    <!-- 响应式网格列数 -->
  </div>
  
  <!-- ✅ 隐藏/显示 -->
  <div class="hidden md:block">桌面端显示</div>
  <div class="block md:hidden">移动端显示</div>
</template>
```

### 交互状态
```vue
<template>
  <!-- ✅ Hover 状态 -->
  <button class="bg-blue-500 hover:bg-blue-600">悬停变色</button>
  <div class="opacity-80 hover:opacity-100">悬停不透明</div>
  
  <!-- ✅ Focus 状态 -->
  <input class="border border-gray-300 focus:border-blue-500 focus:ring-2 focus:ring-blue-200">
  
  <!-- ✅ Active 状态 -->
  <button class="active:scale-95">点击缩放</button>
  
  <!-- ✅ Disabled 状态 -->
  <button class="disabled:opacity-50 disabled:cursor-not-allowed">禁用按钮</button>
</template>
```

### 过渡动画
```vue
<template>
  <!-- ✅ 过渡效果 -->
  <div class="transition-all duration-300 ease-in-out">
    全部属性过渡
  </div>
  
  <div class="transition-colors duration-200">
    颜色过渡
  </div>
  
  <div class="transition-transform duration-300 hover:scale-110">
    悬停放大
  </div>
  
  <!-- ✅ 动画 -->
  <div class="animate-spin">旋转动画</div>
  <div class="animate-pulse">脉冲动画</div>
  <div class="animate-bounce">弹跳动画</div>
</template>
```

## Less 使用规范

### 变量定义
```less
// styles/variables.less

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jianxcao/transmission-web](https://github.com/jianxcao/transmission-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
