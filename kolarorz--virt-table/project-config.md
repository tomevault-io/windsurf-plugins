---
trigger: always_on
description: > 本文件为 AI Agent / 后续开发者提供项目全景与功能上下文。阅读本文即可快速定位代码、理解架构、明确已实现能力与待补齐能力。
---

# AGENTS.md — virt-table 开发上下文

> 本文件为 AI Agent / 后续开发者提供项目全景与功能上下文。阅读本文即可快速定位代码、理解架构、明确已实现能力与待补齐能力。

## 1. 项目定位

`virt-table` 是一套**高性能虚拟滚动表格**，核心卖点是**行 + 列双向虚拟化**，可承载十万级数据流畅滚动。提供框架无关的 Vanilla 实现，并封装出 React / Vue 组件。

- 语言：TypeScript（ESM，`"type": "module"`）
- 包管理：pnpm workspace（`pnpm@9.15.2`）
- 构建：Vite（库模式，多入口 ESM + CJS，另加一轮 UMD；见 §3.2）；文档站：VitePress
- 测试：Vitest。纯逻辑用默认 node 环境（`merge` / `pipeline` / `summary` / `export` / `filter-model` / `header-group` / `locale` / `pagination` / `plugin` / `remote` / `internal/tooltip`）；DOM 层用 `happy-dom`，**按文件开启**（`// @vitest-environment happy-dom` docblock），见 §5.1

## 2. 仓库结构

```
virt-table/
├─ packages/               # 发布包（真正的源码）
│  ├─ vanilla/             # @virt-table/vanilla —— 核心实现（3200+ 行）
│  │  └─ src/
│  │     ├─ index.ts       # VirtTable 主类：所有表格逻辑
│  │     ├─ merge.ts       # 合并单元格算法（含 merge.test.ts 1600+ 行测试）
│  │     ├─ header-group.ts # 多级分组表头/表尾（列树 → 网格 + 横向虚拟化计划）
│  │     ├─ spreadsheet.ts # 电子表格模式：单元格类型 / 剪贴板序列化
│  │     ├─ plugin.ts      # 插件契约 + PluginRuntime（含 plugin.test.ts）
│  │     ├─ index.css      # 核心样式 → 产物 dist/core.css（可选层样式各自成文件）
│  │     ├─ umd.ts         # 仅 UMD 构建用的全量入口（不进 exports）
│  │     ├─ plugins/       # 插件：可选 UI/交互层，各自带 css，入口 /plugins
│  │     │  ├─ index.ts                # barrel：@virt-table/vanilla/plugins
│  │     │  ├─ _panel.css              # 通用面板样式，被下面三方 CSS @import 内联
│  │     │  ├─ context-menu.ts / .css   # vtContextMenu
│  │     │  ├─ export.ts               # vtExport（导出 CSV/Excel + 打印，无 CSS）
│  │     │  ├─ clipboard.ts            # vtClipboard（选区复制/粘贴，无 CSS）
│  │     │  ├─ search.ts / search.css   # vtSearch（Ctrl/Cmd+F）
│  │     │  ├─ column-panel.ts / .css   # vtColumnPanel（列设置面板）
│  │     │  ├─ cell-selection.ts / .css # vtCellSelection（框选，provide cellSelection）
│  │     │  ├─ keyboard-nav.ts         # vtKeyboardNav（requires cellSelection，无 CSS）
│  │     │  ├─ cell-editor.ts / .css    # vtCellEditor（编辑浮层，provide cellEditor）
│  │     │  ├─ column-filter.ts / .css  # vtColumnFilter（列头筛选下拉）
│  │     │  └─ drag-sort.ts / .css      # vtColumnDrag + vtRowDrag
│  │     ├─ internal/      # 多方共享的手势/滚动基元（不对外导出）
│  │     │  ├─ drag.ts                 # startDrag / swallowNextClick（resize + 两个拖拽插件）
│  │     │  ├─ auto-scroll.ts          # 边缘自动滚动（框选 + 行拖拽）
│  │     │  └─ tooltip.ts / .test.ts   # textOverflow:'tooltip' 的浮层（定位是纯函数，有单测）
│  │     └─ components/    # 内置单元格组件（工厂函数，见 §7.14 / §7.17），入口 /components
│  │        ├─ index.ts                # barrel：@virt-table/vanilla/components
│  │        ├─ shared.ts               # 公共类型 + commitValue / stopCellInterference / 锁高容器
│  │        ├─ _popup.ts               # 自绘浮层基元（挂 .vt-client + data-vt-popup，见 §7.14）
│  │        ├─ 编辑态：vt-input / vt-number-input / vt-textarea / vt-select /
│  │        │          vt-multi-select / vt-cascader / vt-autocomplete /
│  │        │          vt-date-picker / vt-date-range-picker / vt-time-picker
│  │        ├─ 直接可点：vt-checkbox / vt-switch / vt-rate / vt-actions
│  │        ├─ 纯展示：vt-tag / vt-progress / vt-link
│  │        ├─ vt-filter-builder.ts / .css  # 单独成入口（CSS 416 行）
│  │        ├─ components.css          # .vt-comp-* 轻 DOM 外观（@import 共享 _panel.css）
│  │        └─ components-contract.test.ts / components.dom.test.ts
│  │     ├─ class-contract.test.ts  # JS 写出的 class ↔ CSS 选择器（见 §5.3）
│  ├─ react/               # @virt-table/react —— 薄封装（273 行）
│  └─ vue/                 # @virt-table/vue —— 薄封装（252 行）
├─ docs/                   # VitePress 文档站 + 三端 playground（apps/{react,vue,vanilla}）
│  ├─ vanilla|react|vue/   # 各端 guide / examples / api 文档
├─ package.json            # workspace 根，脚本入口
└─ tsconfig.base.json
```

> ⚠️ **注意**：README 中描述的 `packages/core`、`packages/dom` 目录**当前不存在**。纵向虚拟滚动内核来自**外部依赖 `@virt-list/core`**（见 `node_modules/@virt-list/core`），不在本仓库维护。本仓库只维护 `vanilla / react / vue` 三个包。
>
> 该依赖**从 npm 安装**（`^0.0.6`，声明在根 `package.json` 与 `packages/vanilla/package.json`），不再是指向兄弟仓库的 `link:`。要联调上游改动，用 `pnpm overrides` 或临时 `pnpm link`，别把 `link:` 写回 `package.json`。

## 3. 架构分层

```
┌──────────────────────────────────────────────┐
│         React / Vue 适配层 (薄封装)            │  packages/{react,vue}/src/index.ts
│  VNode/JSX ↔ DOM 桥接、生命周期、列转换         │  依赖 react-dom createRoot / vue render
├──────────────────────────────────────────────┤
│         插件层（可选，按需引入）                │  packages/vanilla/src/plugins/*
│  11 个插件：框选/键盘/编辑/筛选/拖拽/搜索/…      │  契约在 src/plugin.ts
├──────────────────────────────────────────────┤
│         VirtTable (vanilla) —— 核心            │  packages/vanilla/src/index.ts
│  列管理 | 固定列 | 横向虚拟化 | 合并单元格      │
│  树形/分组/展开 | 排序 | 筛选谓词 | 列宽拖拽     │
│  多级表头/表尾 | 单元格几何 | 列/行重排          │
├──────────────────────────────────────────────┤
│         VirtListCore (外部 @virt-list/core)    │  node_modules
│  纵向虚拟滚动引擎（框架无关，不操作 DOM）        │
└──────────────────────────────────────────────┘
```

**关键设计原则**：
- 所有 2D 表格逻辑集中在 vanilla 层，React/Vue **不各自重复实现**，仅做框架 VNode → DOM 挂载与回收（行被虚拟滚动回收时清理框架组件树，见 `onRowRemoved`）。
- `VirtListCore` 只负责一维纵向窗口计算（`ListState`：`itemsTotalSize / leadingSize / inViewBegin / renderBegin / renderEnd` 等）。
- 横向列虚拟化由 vanilla 自己实现（`_calcColRange` / `_buildColPrefixSums` / `_onHScroll` / `colBuffer`）。

**DOM 结构**（简化）：

```
.vt-root
  ├─ .vt-client (overflow-x:auto; overflow-y:hidden; tabindex=0)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kolarorz/virt-table](https://github.com/kolarorz/virt-table) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
