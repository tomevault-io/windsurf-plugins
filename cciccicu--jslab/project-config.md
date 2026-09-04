---
trigger: always_on
description: > 本文档基于 [Xiaomi Vela JS 应用开发文档](./VelaDocs/docs/zh/guide/) 整理，专为智能可穿戴设备应用开发设计。
---

# AGENTS.md - Vela JS 应用开发指南

> 本文档基于 [Xiaomi Vela JS 应用开发文档](./VelaDocs/docs/zh/guide/) 整理，专为智能可穿戴设备应用开发设计。  
> **固定适配尺寸：width: 336px; height: 480px**（小米手环 Pro 分辨率）
> 文档检索工具：./VelaDocs/search.py，用法：python search.py "Vela OS"

---

## 重要提示：这不是 HTML/EJS 模板开发

**Vela JS 应用使用专有的 UX 语法，而非标准 HTML 或 EJS 等模板引擎。** 虽然语法可能与 HTML 相似，但 Vela 拥有自己的组件系统、渲染引擎和开发范式。

---

## Vela JS 架构概述

Vela JS 应用是基于小米 Vela OS 的轻量级应用模型，专为内存和处理能力有限的智能可穿戴设备设计。

### 核心特性

- **轻量级**：更小的体积，便于在可穿戴设备上快速加载和运行
- **跨平台**：一次开发，多端运行
- **高性能**：优化渲染能力，支持 60fps 流畅体验
- **安全性**：三重隔离机制，保障数据与设备安全

### 应用场景

- 健康监测：实时监测心率、睡眠质量等健康数据
- 运动辅助：记录运动数据，提供运动指导
- 消息提醒：显示手机等设备的通知消息
- 智能控制：作为智能家居控制中心远程操控设备
- 日常工具：提供天气预报、闹钟、计时器等工具功能

---

## 项目结构

```
├── manifest.json          # 项目配置文件
├── app.ux                 # 应用生命周期与全局数据
├── pages/
│   ├── index/
│   │   └── index.ux       # 页面文件，包含 template/style/script
│   └── detail/
│       └── detail.ux
├── i18n/                  # 国际化文件
├── common/                # 共享资源（图片、样式、工具函数）
└── resources/             # 应用资源文件
```

[项目结构详解](./VelaDocs/docs/zh/guide/framework/project-structure.md)

---

## UX 文件结构

每个 `.ux` 文件包含三个必需部分：

```html
<template>
  <!-- 使用 Vela 组件编写页面结构 -->
</template>

<style>
  /* 类 CSS 样式，用于组件样式定义 */
</style>

<script>
  // JavaScript 逻辑，控制页面行为
  export default {
    private: {
      // 页面数据
    },
    onInit() {
      // 生命周期钩子
    }
  }
</script>
```

[UX 文件说明](./VelaDocs/docs/zh/guide/framework/ux.md)

---

## 固定尺寸配置（336x480）

### manifest.json 配置

```json
{
  "package": "com.example.demo",
  "name": "示例应用",
  "icon": "/Common/icon.png",
  "versionName": "1.0",
  "versionCode": 1,
  "minAPILevel": 1,
  "features": [
    { "name": "system.router" },
    { "name": "system.fetch" }
  ],
  "config": {
    "designWidth": 336
  },
  "router": {
    "entry": "index",
    "pages": {
      "index": {
        "component": "index",
        "path": "/"
      }
    }
  },
  "display": {
    "backgroundColor": "#000000"
  }
}
```

### 样式基准

- **设计基准宽度**：336px
- **设计基准高度**：480px
- **长度单位**：px（相对于 designWidth）、%、dp
- **布局系统**：仅支持 Flex 布局

[页面样式与布局](./VelaDocs/docs/zh/guide/framework/style/page-style-and-layout.md)

---

## 支持的组件

### 基础组件

| 组件 | 说明 | 文档 |
|------|------|------|
| `text` | 文本展示（所有文本必须使用此组件） | [text 组件](./VelaDocs/docs/zh/components/basic/text.md) |
| `span` | 格式化文本，仅可作为 text/a/span 的子组件 | [span 组件](./VelaDocs/docs/zh/components/basic/span.md) |
| `image` | 图片展示，支持 png/jpg 格式 | [image 组件](./VelaDocs/docs/zh/components/basic/image.md) |
| `progress` | 进度条，支持 horizontal/arc 类型 | [progress 组件](./VelaDocs/docs/zh/components/basic/progress.md) |
| `qrcode` | 二维码生成与展示 | [qrcode 组件](./VelaDocs/docs/zh/components/basic/qrcode.md) |
| `barcode` | 条形码生成与展示（Code128 码） | [barcode 组件](./VelaDocs/docs/zh/components/basic/barcode.md) |

### 容器组件

| 组件 | 说明 | 文档 |
|------|------|------|
| `div` | 基础容器，用作根节点或内容分组 | [div 组件](./VelaDocs/docs/zh/components/container/div.md) |
| `list` | 列表视图容器，仅支持 list-item 子组件 | [list 组件](./VelaDocs/docs/zh/components/container/list.md) |
| `list-item` | 列表项组件，必须设置 type 属性 | [list-item 组件](./VelaDocs/docs/zh/components/container/list-item.md) |
| `scroll` | 滚动视图容器，支持横向/纵向滚动 | [scroll 组件](./VelaDocs/docs/zh/components/container/scroll.md) |
| `stack` | 层叠容器，子组件按顺序堆叠 | [stack 组件](./VelaDocs/docs/zh/components/container/stack.md) |
| `swiper` | 滑块视图容器，支持自动播放/循环 | [swiper 组件](./VelaDocs/docs/zh/components/container/swiper.md) |

### 表单组件

| 组件 | 类型 | 说明 | 文档 |
|------|------|------|------|
| `input` | button/checkbox/radio | 用户输入与选择 | [input 组件](./VelaDocs/docs/zh/components/form/input.md) |
| `picker` | text/time | 滚动选择器 | [picker 组件](./VelaDocs/docs/zh/components/form/picker.md) |
| `switch` | - | 开关选择 | [switch 组件](./VelaDocs/docs/zh/components/form/switch.md) |
| `slider` | - | 滑动选择器 | [slider 组件](./VelaDocs/docs/zh/components/form/slider.md) |

[组件总览](./VelaDocs/docs/zh/components/)

---

## 语法规范

### 数据绑定

```html
<template>
  <text>{{message}}</text>
</template>

<script>
export default {
  private: {
    message: 'Hello'
  }
}
</script>
```

[数据绑定文档](./VelaDocs/docs/zh/guide/framework/template/#数据绑定)

### 事件绑定

支持常规写法与简写语法：

```html
<template>
  <div>
    <text onclick="press"></text>
    <text @click="press"></text>
  </div>
</template>

<script>
export default {
  press(e) {
    this.title = 'Hello'
  }
}
</script>
```

事件回调支持语法：
- `fn`：函数名，`<script>` 中需有对应实现
- `fn(a,b)`：参数可为常量或 `<script>` 中定义的变量
- 回调时自动追加 `evt` 参数，可访问事件上下文数据

[事件绑定文档](./VelaDocs/docs/zh/guide/framework/template/event.md)

### 列表渲染

```html
<template>
  <div>
    <div for="{{list}}" tid="uniqueId">
      <text>{{$idx}}</text>
      <text>{{$item.uniqueId}}</text>
    </div>
  </div>
</template>

<script>
export default {
  private: {
    list: [
      { uniqueId: 1 },
      { uniqueId: 2 }
    ]
  }
}
</script>
```

`for` 指令支持语法：
- `for="{{list}}"`：默认元素名为 `$item`
- `for="{{value in list}}"`：自定义元素名，默认索引名为 `$idx`
- `for="{{(index, value) in list}}"`：自定义索引名与元素名

`tid` 属性注意事项：
- 指定的数据属性必须存在且唯一
- 当前不支持表达式
- 用于复用节点，优化重绘效率

[列表渲染文档](./VelaDocs/docs/zh/guide/framework/template/for.md)

### 条件渲染

两类方式：`if/elif/else` 与 `show`

- `if` 为 `false` 时组件从 VDOM 移除
- `show` 为 `false` 时组件仅不可见，仍存在于 VDOM

```html
<template>
  <div>
    <text if="{{display}}">Hello-1</text>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cciccicu/JSLab](https://github.com/cciccicu/JSLab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
