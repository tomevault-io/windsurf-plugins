---
trigger: always_on
description: > 这个文件是给 Codex 看的项目状态说明,任何新的 Codex 会话开始前都必须读完。
---

# aiditor — Codex 工作交接

> 这个文件是给 Codex 看的项目状态说明,任何新的 Codex 会话开始前都必须读完。
> 用户在不同电脑之间切换工作环境,本文件保证上下文不丢失。

---

## 1. 项目是什么

**aiditor** —— 一个纯前端、零依赖、Blender 风格的通用编辑器框架。

当前产品边界分四块:

1. **AIditor Core/UI**:稳定零依赖内核,提供 Dock/Panel/Component、UI 组件库、主题、signal/log/bus/settings/history/workspace contract。
2. **AIditor AI Host**:可选上层模块,提供 agent runtime、provider、tools/context/operations、permissions、ChangeSet、compaction。它依赖 Core/UI,但 Core/UI 不依赖 AI。
3. **AIditor Extension Runtime**:可选上层模块,把 component/tool/context/reference/operation/settings/command/menu/dock panel contribution 安装进已有 registry。它不是第二套组件或 AI 模型。
4. **Demo Project Runtime**:示例宿主应用,用于演示“打开 workspace、加载文件、注册组件、挂到 dock”。它不属于框架层概念,不得写进 `src/` 的通用设计。

AIditor 仍坚持零依赖、零模块系统、单命名空间和 file:// 可运行。AI Host 和 Extension Runtime 是框架提供的可选能力,不是把 Core/UI 变成业务编辑器。

- **零构建**:经典 `<script>` 标签,直接 `file://` 双击 `index.html` 就能跑
- **零依赖**:不用 npm,不用打包工具,不用任何框架
- **单命名空间**:所有东西挂在 `window.aiditor` 下
- **三层架构**:Dock(布局容器)/ Panel(内容单元)/ Component(UI 组件)
- **核心机制**:不可变 N 叉分割树 + 自研 ~70 行响应式 signal + 按 dock id 的 keyed reconciliation

### 核心思想(读完这一段等于看懂了一半)

整个框架就是这一张图:

```
Layout(Blender 风格的 N 叉分割树)
 └─ Dock ×M                      ← 可以被分裂 / 合并 / 调整大小的矩形容器
     ├─ Toolbar(可选,top/bottom/left/right 四选一条)
     │   ├─ Static Items          ← dock 配置写死的(方向、初始按钮等)
     │   └─ Dynamic Items         ← active panel 动态贡献的,切 active 自动装卸
     └─ Panel ×N(同一时刻只有 1 个 active,也可能 0 个)
         └─ Component(真正渲染内容的 UI 组件)
```

**关键结论**(每一条都是一条硬约束,后面章节会细化):

1. **Dock 里装 0 ~ N 个 panel**,有 panel 时**总有一个 active**,只 active 的那个显示。
2. **Dock 的显示 = toolbar + panel content**,toolbar 可无,content 区永远存在(空 dock 时是空 div)。
3. **Toolbar 组件不分种类**:`tab-standard`、"关闭按钮"、"文件切换器"全都是平等的 toolbar 组件。tab component 的"特殊性"仅仅是它订阅了所在 dock 的 `panels` 和 `activeId` signal 从而能渲染成 tab 栏 —— 但框架不给它开任何特权 API,它用的 ctx 和别的 toolbar 组件完全一样。
4. **Toolbar 组件有两种来源**:
   - **Static**:dock 配置里写的,随 dock 生命周期存在
   - **Dynamic**:active panel 在 PanelData 里声明的 `toolbarItems[]`,panel 激活时自动挂到 toolbar,切走自动卸载。panel 跨 dock 移动时,动态 items 自然跟着 panel 走。
5. **Panel 可跨 dock 拖放,也可弹独立窗口**。Dock 可以配 `accept` 白名单限定只接受哪些 component 类型的 panel。跨 dock 拖放和同 dock 切 active 共享同一条 detach/re-attach 代码路径;跨窗口则走 `serialize`/`deserialize` 协议(§ 4.14)。
6. **多 panel 的性能要求是真的"只有 active 存在"**:非 active panel 的 contentEl **直接从 DOM detach**(不是 display:none,不是 content-visibility:hidden),浏览器对它零 layout、零 paint、零事件开销。切回 active 时 re-append,DOM 状态和 JS 对象完全保留。这是 § 4.3 的唯一实现路径。
7. **Panel / Dock 之间通讯走一条统一的解耦总线 `aiditor.bus`**:pub/sub,topic + payload,通过 `ctx.bus` 自动在 panel dispose 时取消订阅。没人直接持有别人的引用。

本文件是**最高优先级的工作交接与硬规则权威**。当前架构细节以 `doc/*.md` 为准,但不得违反本文件里的产品边界、零依赖、零模块系统、设计先行和代码风格红线。`doc/old/**` 只作历史资料。

---

## 2. 硬规则(不可违反)

这些是和用户多次对话后确立的红线,违反会让用户失望。

### 2.1 零应用级快捷键
**框架代码绝对不许内置"应用级/业务级"快捷键**。
我们是通用库,不是某个特定的编辑器应用。Focus Mode、关闭 panel、切换 tab、保存、命令面板、撤销/重做……所有"属于应用决策"的快捷键都**只暴露 API**(例如 `ctx.dock.toggleFocus()`),由调用方决定要不要绑键、绑哪个键。Demo 里可以演示一种绑法,但绝不写进 `src/`。

**但"组件内部语义键"允许,且必须有**——这不是快捷键,是组件自身功能的一部分,删掉组件就废了:
- **输入/编辑组件的编辑键**:textarea 的 Tab 缩进、codeInput 的 Tab、input 的 Enter 提交等。浏览器默认行为不满足组件语义时,组件必须自己 `preventDefault` + 处理
- **Overlay 的 dismiss 键**:modal / drawer / popover / menu 按 ESC 关闭最上层(由 `_overlay.js` 统一管,LIFO 栈)
- **Focus trap 的 Tab 循环**:modal 打开时 Tab 在 modal 内部循环,防止焦点跑到背后不可见元素。这是 WAI-ARIA 对 modal dialog 的硬性要求
- **进行中的交互的取消键**:拖拽 splitter / 拖 panel 过程中按 ESC 取消本次拖拽

判据很简单:**这个键绑了之后,是在替用户决定应用该怎么响应,还是在完成组件自己必须做的事?** 前者禁止(写 API 让调用方绑),后者允许(组件自己绑)。拿不准就当作前者。

### 2.2 零构建、零模块系统
**不许用 ES modules,不许引入打包工具,不许写 `import/export`。**
所有源文件都是 IIFE,挂载到 `window.aiditor`:
```js
;(function (aiditor) {
  'use strict'
  // ...
  aiditor.something = something
})(window.aiditor = window.aiditor || {})
```
HTML 用 `<script src="...">` 按依赖顺序加载。用户必须能双击 `index.html` 直接看到运行效果。

### 2.3 设计先行(Design-First)
**任何非平凡的改动,先写计划,等用户明确说"开始"再动代码。**
顺序:
1. 列数据模型 / 文件清单 / API 表面
2. 列出待决问题并明确请用户拍板
3. 用户回复后修订计划,可能多轮
4. 用户回复"开始" / "go" / "确认开始"才动代码

用户在动代码之前更正过设计方向多次。如果你跳过这一步直接动手,会浪费工作量。当用户说"先不着急改代码"或类似的话,意思就是只设计不写代码。

### 2.4 一个独立功能一个文件
独立的功能单元住在独立的文件里。`src/` 下用子目录把相关关注点分组(`core/`、`tree/`、`components/`、`dock/`、`style/`)。不要把 6 个不相关的概念塞进一个 800 行的文件。但也不要把 30 行的"焦点模式"硬拆出去 —— 见 § 5 的目录方案。

### 2.5 不写防御性代码
框架内部相互调用是受信任的契约,不需要 try/catch、null 检查、参数兜底。**只有在用户 component 调用边界用 `safeCall` 包裹**(因为用户代码可能抛错)。不为不可能发生的情况写代码。

### 2.6 不擅自加功能
- 没让你做的功能不要做("顺手清理一下"、"加点配置项"、"补个 docstring"全都不要)
- 没让你重构的代码不要重构
- 修 bug 时不连带改无关代码
- 不在没改的代码上加注释 / 类型 / 文档
- 不为假想的未来需求做准备

### 2.7 不擅自破坏性操作
不未经允许:`git push --force`、`git reset --hard`、删文件、改 git 配置、`--no-verify`。

---

## 3. 当前代码状态(已实现)

> 本节是当前实现快照。读完这一节就知道代码到什么程度了;不要用旧对话、旧阶段或 `doc/old/**` 推断当前架构。

### 3.1 目录(实际落盘)

```
aiditor/
  index.html                       # demo 入口 — 引用 dist/aiditor-full.{css,js} + demo widgets
  AGENTS.md                        # 本文件 — 工作交接与硬规则权威
  doc/
    old/editor_style.html          # 视觉调色板历史参考(只读,不改)

  tools/
    build.mjs                      # § 2.2 零构建承诺的载体:cat 带 banner,
                                   # 拼出轻量切片和 classic bundles;支持 --watch

  dist/                            # 已 commit 的 bundle 产物(保证零环境双击运行)
    aiditor-theme.js / .css        # 独立主题 runtime + tokens + 内置主题
    aiditor-mini.js / .css         # 独立网页常用 UI + 主题,不含编辑器重组件
    aiditor-editor.js / .css       # 独立完整通用编辑器 UI,不含 Dock/AI
    aiditor-kernel.js / .css       # Core services + tree + Dock runtime

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jinruozai/aiditor](https://github.com/jinruozai/aiditor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
