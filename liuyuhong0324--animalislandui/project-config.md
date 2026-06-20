---
trigger: always_on
description: >
---


# Animal Island UI 设计风格指南

> **三文档分工**（生成代码 / 调样式时按需查阅，避免互相翻查）：
> - `AI_USAGE.md` — API 手册：每个组件的 props、类型、默认值、合法取值、禁用用法。**写代码优先查这里**。
> - `skill/SKILL.md`（本文档）— 像素级样式：设计 token、每组件精确 CSS（hex/px/keyframe）、Demo 布局、新组件开发模板。**要自己实现/扩展样式时查这里**。
> - `DESIGN_PROMPT.md` — 给外部工具（v0 / Figma AI / Midjourney / DALL-E）的提示词包，含 clip-path、色板速查、禁用清单。**只在喂别的 AI 时用**。

## 概述

animal-island-ui 是一套受《集合啦！动物森友会》启发的 React + TypeScript UI 组件库。
设计语言核心：**温暖大地色系 + 大圆角 pill 形 + 游戏按键立体感 + 柔和动效 + 有机不规则形状**。

- 源码：`src/components/<ComponentName>/`
- Demo 站：`demo/`
- 构建：Vite (library mode) + `vite.config.ts`（库）/ `vite.config.demo.ts`（Demo）
- 样式系统：Less Modules + `src/styles/variables.less` 设计 token

### 全量组件清单（17 个）

从 `src/index.ts` 导出：

| 组件 | 职责 | 交互 | 装饰 / 纯展示 |
|---|---|---|---|
| `Button` | 按钮，5 种类型 × 3 种尺寸 | ✓ | |
| `Input` | 输入框，3 种尺寸 + clear/prefix/suffix | ✓ | |
| `Switch` | 开关，默认/小号 | ✓ | |
| `Modal` | SVG blob 裁切弹窗 | ✓ | |
| `Card` | 容器，`default`/`title`，13 种 NookPhone 配色 | | ✓ |
| `Collapse` | 手风琴（CSS Grid，无 JS 过渡） | ✓ | |
| `Select` | 下拉选择器（受控） | ✓ | |
| `Checkbox` | 多选框组，水平/垂直，3 种尺寸 | ✓ | |
| `Icon` | SVG 图标库（10 个） | | ✓ |
| `Time` | HUD 实时时钟 | | ✓ |
| `Phone` | NookPhone 3×3 应用网格 | | ✓ |
| `Footer` | 底部装饰图（`sea`/`tree`） | | ✓ |
| `Divider` | 装饰分割线，5 种风格 | | ✓ |
| `Cursor` | 游戏手指光标包裹器 | | ✓ |
| `Typewriter` | 打字机效果，保留 ReactNode 结构 | | ✓ |
| `Tabs` | 标签页切换，叶子摆动动画可选 | ✓ | |
| `CodeBlock` | JSX/TS 语法高亮代码块 | | ✓ |

类型导出：`ButtonProps/ButtonType/ButtonSize`、`InputProps/InputSize`、`SwitchProps/SwitchSize`、`ModalProps`、`CardProps/CardType/CardColor`、`FooterProps/FooterType`、`CollapseProps`、`CursorProps`、`TimeProps`、`PhoneProps`、`DividerProps`、`TypewriterProps`、`SelectProps/SelectOption`、`IconProps/IconName`、`TabsProps/TabItem`、`CheckboxProps/CheckboxOption/CheckboxSize`、`CodeBlockProps`。运行时值：`ICON_LIST`。

---

## 1. Design Tokens

### 色彩系统

```less
// 主色（薄荷青绿）
@primary-color:        #19c8b9;
@primary-color-hover:  #3dd4c6;
@primary-color-active: #11a89b;
@primary-color-bg:     #e6f9f6;

// 文字（温暖棕色系）
@text-color:           #794f27;    // 主文字（header/sidebar）
@text-color-body:      #725d42;    // 正文（组件内文字）
@text-color-secondary: #9f927d;    // 次级文字
@text-color-muted:     #8a7b66;    // 浅棕（modal body）
@text-color-disabled:  #c4b89e;    // 禁用

// 边框
@border-color:         #9f927d;
@border-color-light:   #c4b89e;    // 输入框边框
@border-color-hover:   #a89878;    // 输入框 hover

// 背景（奶油米白）
@bg-color:             #f8f8f0;    // 主背景
@bg-color-content:     rgb(247, 243, 223);  // 内容区（Modal、Card）
@bg-color-secondary:   #f0e8d8;
@bg-color-disabled:    #f0ece2;
@bg-color-input:       rgb(247, 243, 223);  // 输入框背景
@bg-color-input-dis:   #ece8dc;    // 输入框禁用

// 状态色
@success-color:        #6fba2c;
@success-color-active: #5a9e1e;
@warning-color:        #f5c31c;
@warning-color-active: #dba90e;
@error-color:          #e05a5a;
@error-color-active:   #c94444;

// 游戏特殊色
@focus-yellow:         #ffcc00;    // 焦点高亮（非蓝色）
@focus-yellow-dark:    #e0b800;    // 焦点阴影
@sidebar-active-bg:    #B7C6E5;    // 侧边栏选中背景
@sidebar-hover-bg:     #d6dff0;    // 侧边栏 hover 背景

// 3D 阴影色
@shadow-btn:           #bdaea0;    // 按钮 3D 阴影
@shadow-input:         #d4c9b4;    // 输入框 3D 阴影
@shadow-switch-on:     #5a9e1e;    // Switch 开启 3D 阴影
```

**NookPhone 应用调色板**（Card `color` prop 可选值）：

| color 值 | 背景色 | 文字色 |
|---|---|---|
| default | `rgb(247, 243, 223)` | `#725d42` |
| app-pink | `#f8a6b2` | `#fff` |
| purple | `#b77dee` | `#fff` |
| app-blue | `#889df0` | `#fff` |
| app-yellow | `#f7cd67` | `#725d42` |
| app-orange | `#e59266` | `#fff` |
| app-teal | `#82d5bb` | `#fff` |
| app-green | `#8ac68a` | `#fff` |
| app-red | `#fc736d` | `#fff` |
| lime-green | `#d1da49` | `#3d5a1a` |
| yellow-green | `#ecdf52` | `#725d42` |
| brown | `#9a835a` | `#fff` |
| warm-peach-pink | `#e18c6f` | `#fff` |

---

### 字体

项目使用三款 Google Fonts 圆体字，**必须**按以下方式引入，本地未安装时通过在线地址加载：

```html
<!-- 在 index.html <head> 中引入 -->
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
<link
  href="https://fonts.googleapis.com/css2?family=Nunito:wght@400;500;600;700;800;900&family=Zen+Maru+Gothic:wght@400;500;700&family=M+PLUS+Rounded+1c:wght@400;500;700&display=swap"
  rel="stylesheet"
/>
```

或在 CSS / Less 入口文件顶部：

```css
@import url('https://fonts.googleapis.com/css2?family=Nunito:wght@400;500;600;700;800;900&family=Noto+Sans+SC:wght@400;500;700&family=Zen+Maru+Gothic:wght@400;500;700&display=swap');
```

```css
font-family: Nunito, 'Noto Sans SC', 'Zen Maru Gothic',
  -apple-system, 'PingFang SC', 'Hiragino Sans GB', 'Microsoft YaHei', sans-serif;
```

| 字体 | 用途 | Google Fonts key |
|---|---|---|
| **Nunito** | 主字体，拉丁字符 | `family=Nunito` |
| **Noto Sans SC** | 中文字体，简体覆盖 | `family=Noto+Sans+SC` |
| **Zen Maru Gothic** | 日文字体 | `family=Zen+Maru+Gothic` |

字重分级：
- 正文内容：**500**
- 按钮文字、标题、菜单项：**600–700**
- 数字强调（时间数字、时钟）：**900**
- placeholder / 说明文字：**400**

字间距：`letter-spacing: 0.01em`（正文）/ `0.02em`（按钮/标题）/ `1.5px`（星期大写）

禁止使用细体（weight < 400）或等宽字体。

---

### 间距 / 圆角 / 边框

```
间距：xs=4px  sm=8px  md=12px  lg=16px  xl=24px
圆角：sm=12px  base=18px  lg=24px  pill=50px（按钮/输入框）
边框：默认 2px solid，输入框 2.5px，大尺寸输入框 3px
```

---

### 阴影

```css
/* 卡片/容器阴影（暖色调，非冷黑）*/
box-shadow: 0 3px 10px 0 rgba(61, 52, 40, 0.10);   /* 基础 */
box-shadow: 0 8px 24px 0 rgba(61, 52, 40, 0.14);   /* 较大 */

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liuyuhong0324/AnimalIslandUI](https://github.com/liuyuhong0324/AnimalIslandUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
