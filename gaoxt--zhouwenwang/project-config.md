---
trigger: always_on
description: **必须严格遵守以下规则，禁止使用任何不存在的 className，避免 AI 幻觉生成虚假类名。**
---

# Tailwind CSS + Headless UI 严格约束规则

**必须严格遵守以下规则，禁止使用任何不存在的 className，避免 AI 幻觉生成虚假类名。**

## ⚠️ 核心原则

- **只使用官方文档确认存在的类名**
- **禁止使用小数点数值**（如 `gap-2.5`、`p-1.5` 等）
- **禁止自创组合类名**
- **所有类名必须经过验证**

## 🎯 Tailwind CSS 间距系统（Spacing Scale）

### ✅ 允许的间距值（Tailwind v4.1）

**基础间距单位：`--spacing: 0.25rem` (4px)**

```
0     = 0px
px    = 1px
0.5   = 2px     (0.125rem)
1     = 4px     (0.25rem)
1.5   = 6px     (0.375rem)
2     = 8px     (0.5rem)
2.5   = 10px    (0.625rem)
3     = 12px    (0.75rem)
3.5   = 14px    (0.875rem)
4     = 16px    (1rem)
5     = 20px    (1.25rem)
6     = 24px    (1.5rem)
7     = 28px    (1.75rem)
8     = 32px    (2rem)
9     = 36px    (2.25rem)
10    = 40px    (2.5rem)
11    = 44px    (2.75rem)
12    = 48px    (3rem)
14    = 56px    (3.5rem)
16    = 64px    (4rem)
20    = 80px    (5rem)
24    = 96px    (6rem)
28    = 112px   (7rem)
32    = 128px   (8rem)
36    = 144px   (9rem)
40    = 160px   (10rem)
44    = 176px   (11rem)
48    = 192px   (12rem)
52    = 208px   (13rem)
56    = 224px   (14rem)
60    = 240px   (15rem)
64    = 256px   (16rem)
72    = 288px   (18rem)
80    = 320px   (20rem)
96    = 384px   (24rem)
```

### ❌ 禁止使用的间距值

```
// 🚫 错误 - 不存在的小数点间距
gap-2.5
p-1.5
m-3.5
space-x-4.5

// 🚫 错误 - 不存在的数值
gap-15
p-17
m-13
```

### ✅ 正确的间距类名

```jsx
// ✅ 正确 - 使用存在的间距值
<div className="gap-2 p-4 m-3 space-x-6">
<div className="px-4 py-2 mx-auto">
<div className="mt-8 mb-4 ml-2 mr-6">
```

## 📏 Gap 系统规则

### ✅ 允许的 Gap 类名

```css
gap-0, gap-px, gap-0.5, gap-1, gap-1.5, gap-2, gap-2.5, gap-3, gap-3.5, gap-4, gap-5, gap-6, gap-7, gap-8, gap-9, gap-10, gap-11, gap-12, gap-14, gap-16, gap-20, gap-24, gap-28, gap-32, gap-36, gap-40, gap-44, gap-48, gap-52, gap-56, gap-60, gap-64, gap-72, gap-80, gap-96

gap-x-0, gap-x-px, gap-x-0.5... (同上数值)
gap-y-0, gap-y-px, gap-y-0.5... (同上数值)
```

### ❌ 禁止的 Gap 用法

```jsx
// 🚫 错误 - 不存在
<div className="gap-2.5"> // Tailwind 中不存在 gap-2.5
<div className="gap-15">  // 不存在 gap-15
<div className="gap-13">  // 不存在 gap-13
```

### ✅ 正确的 Gap 用法

```jsx
// ✅ 正确
<div className="flex gap-4">
<div className="grid grid-cols-3 gap-6">
<div className="flex gap-x-4 gap-y-8">
```

## 🎨 Tailwind CSS 颜色系统

### ✅ 标准颜色等级

**每种颜色都有以下等级：**
```
50, 100, 200, 300, 400, 500, 600, 700, 800, 900, 950
```

### ✅ 允许的颜色类名

```css
/* 基础颜色 */
black, white

/* 灰色系 */
slate-50, slate-100, ..., slate-950
gray-50, gray-100, ..., gray-950
zinc-50, zinc-100, ..., zinc-950
neutral-50, neutral-100, ..., neutral-950
stone-50, stone-100, ..., stone-950

/* 彩色系 */
red-50, red-100, ..., red-950
orange-50, orange-100, ..., orange-950
amber-50, amber-100, ..., amber-950
yellow-50, yellow-100, ..., yellow-950
lime-50, lime-100, ..., lime-950
green-50, green-100, ..., green-950
emerald-50, emerald-100, ..., emerald-950
teal-50, teal-100, ..., teal-950
cyan-50, cyan-100, ..., cyan-950
sky-50, sky-100, ..., sky-950
blue-50, blue-100, ..., blue-950
indigo-50, indigo-100, ..., indigo-950
violet-50, violet-100, ..., violet-950
purple-50, purple-100, ..., purple-950
fuchsia-50, fuchsia-100, ..., fuchsia-950
pink-50, pink-100, ..., pink-950
rose-50, rose-100, ..., rose-950
```

### ❌ 禁止的颜色用法

```jsx
// 🚫 错误 - 不存在的颜色等级
<div className="bg-blue-150">   // 不存在
<div className="text-red-75">   // 不存在
<div className="border-green-550"> // 不存在
```

## 🔤 字体大小系统

### ✅ 允许的字体大小类名

```css
text-xs     = 0.75rem
text-sm     = 0.875rem
text-base   = 1rem
text-lg     = 1.125rem
text-xl     = 1.25rem
text-2xl    = 1.5rem
text-3xl    = 1.875rem
text-4xl    = 2.25rem
text-5xl    = 3rem
text-6xl    = 3.75rem
text-7xl    = 4.5rem
text-8xl    = 6rem
text-9xl    = 8rem
```

### ❌ 禁止的字体大小

```jsx
// 🚫 错误 - 不存在
<h1 className="text-xxl">    // 不存在
<p className="text-medium">  // 不存在
<span className="text-1xl">  // 不存在
```

## 🏗️ Headless UI 组件约束

### ✅ React Headless UI 组件

**@headlessui/react** 包含以下组件：

```jsx
// ✅ 正确的 Headless UI 组件
import {
  Dialog,
  Disclosure,
  Menu,
  Popover,
  RadioGroup,
  Switch,
  Tab,
  Transition,
  Listbox,
  Combobox
} from '@headlessui/react'
```

### ✅ 正确的 Headless UI 用法

```jsx
// ✅ Dialog 组件
<Dialog open={isOpen} onClose={setIsOpen}>
  <Dialog.Panel className="bg-white p-6 rounded-lg">
    <Dialog.Title className="text-lg font-medium">
      Title
    </Dialog.Title>
    <Dialog.Description className="text-gray-500">
      Description
    </Dialog.Description>
  </Dialog.Panel>
</Dialog>

// ✅ Menu 组件
<Menu as="div" className="relative">
  <Menu.Button className="px-4 py-2 bg-blue-500 text-white rounded">
    Options
  </Menu.Button>
  <Menu.Items className="absolute mt-2 bg-white border rounded shadow-lg">
    <Menu.Item>
      {({ active }) => (
        <a className={`block px-4 py-2 ${active ? 'bg-gray-100' : ''}`}>
          Item 1
        </a>
      )}
    </Menu.Item>
  </Menu.Items>
</Menu>
```

### ❌ 禁止的组件使用

```jsx
// 🚫 错误 - 不存在的组件
import { Modal } from '@headlessui/react'     // 应该用 Dialog
import { Dropdown } from '@headlessui/react' // 应该用 Menu
import { Tooltip } from '@headlessui/react'  // Headless UI 没有 Tooltip
```

## 📱 响应式设计约束

### ✅ 标准断点

```css
sm:   = @media (min-width: 640px)
md:   = @media (min-width: 768px)
lg:   = @media (min-width: 1024px)
xl:   = @media (min-width: 1280px)
2xl:  = @media (min-width: 1536px)
```

### ✅ 正确的响应式用法

```jsx

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gaoxt/zhouwenwang](https://github.com/gaoxt/zhouwenwang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
