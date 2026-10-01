---
trigger: always_on
description: 本文件是 `packages/editor`（`@feng3d/editor`）子包的开发规范，**只补充根 [AGENTS.md](../../AGENTS.md) 未覆盖的本包专属内容**。
---

# feng3d-editor 开发规范

本文件是 `packages/editor`（`@feng3d/editor`）子包的开发规范，**只补充根 [AGENTS.md](../../AGENTS.md) 未覆盖的本包专属内容**。
通用规范（纯数据声明式、Logic 类化、响应式 `r_` 前缀、提交规范、测试要求等）一律以根 AGENTS.md 为准，此处不重复。

## 项目概述

feng3d-editor 是基于 feng3d 3D 引擎的可视化编辑器，使用 Vue 3 + TypeScript + Vite 构建。
采用**混合架构**：Vue 3 负责现代 UI 组件，传统 TypeScript 模块负责核心编辑器逻辑。

## 常用命令

```bash
# 开发（必须使用 npm，pnpm 会导致发布失败）
npm run dev

# 构建
npm run build

# 类型检查
npm run type-check

# 代码检查
npm run lint

# 单元测试（vitest，纯逻辑放 test/）
npm run test

# 自动修复代码格式
npm run lintfix

# 清理构建产物
npm run clean
```

## 架构概览

> **功能一律按插件组织**：主界面面板、场景浮层、Logic、属性面板控件、桥接方法都来自插件清单
> （[src/plugins/](src/plugins)），核心只认注册表——加一个面板**不需要改** `MainLayout.vue`。
> 清单是纯数据、注册由 `main.ts` 显式调用（对齐 R2 零模块级副作用）。
> **不要在模块顶层写 `registerLogic` / `setDefaultTypeAttributeView`**——那是会被门禁
> （`scripts/check-editor-module-effects.mjs`）拦下的；加到清单里
> （`contributes.logics` / `contributes.objectView`）即可。
> 插件可**启用/禁用**（设置 → 插件，或桥接 `editor.setPlugin`）：关掉后它的贡献点到处消失
> （面板 / 浮层 / Logic / 属性控件 / 桥接方法），状态持久化、按已安装状态对账。
> 清单**必须**声明 `apiVersion`（不兼容时当场报错，指出"要什么、现在是什么"）；
> 层序是**内置 < 插件 < 用户**，最上层来自本地**不入库**的 `editor.patch.json`
> （模板 `editor.patch.example.json`，坏 patch 不会拖垮编辑器）。
> 详见 [docs/PLUGINS.md](docs/PLUGINS.md)。

### R2 的适用边界（别把"运行时装载"当成副作用）

根 [AGENTS.md](../../AGENTS.md) §15 的 **R2「零模块级副作用」约束的是核心包**：模块**在 import 时**
不得执行代码——禁止模块级 `new Map()` / `new WeakMap()` / `new Set()`、`register*()` 调用、`globalThis` 写入。
本包对应的门禁是 `scripts/check-editor-module-effects.mjs`（除应用入口外，`src/**` 顶层不得有
`registerXxx` / `setDefaultXxx` 调用）。

**边界在于"谁在什么时候触发"**：

| 形态 | 是否违反 R2 | 说明 |
|---|---|---|
| 核心模块顶层调 `registerLogic(...)` / `setDefaultTypeAttributeView(...)` | **违反** | import 即产生隐式副作用，且无法 tree-shake |
| 应用入口显式调用（`src/vue-app/main.ts` 的 `installBuiltinPlugins()`） | 不违反 | 显式安装点，登记在门禁脚本的入口白名单里，脚本会**反向校验**该登记是否过期 |
| **宿主在运行时装载插件**（cordis loader `import()` 插件包并注册其贡献点） | **不违反** | 宿主**装载插件本来就是运行时行为**，且是**显式注册**——这正是 R2 的**边界之外**，不是 R2 要禁止的东西 |
| 插件 runtime 端（游戏项目端）被打进产物 | **仍须遵守** | 产物要可 tree-shake → runtime 端同样不得有模块级副作用 |

一句话：**R2 管的是"核心包在 import 时的隐式副作用"，不是"运行时不许注册"**。
（这条边界是 [docs/PLUGINS.md](docs/PLUGINS.md) 的 cordis 结论得以反转的前提。）

### 双架构设计

1. **传统 UI 层**（[src/ui/](src/ui)）
   - 早期基于自定义系统的 UI 组件
   - 包括 hierarchy（层级树）、inspector（属性检查器）、assets（资源管理）
   - 直接操作 DOM，不使用 Vue

2. **Vue UI 层**（[src/vue-app/](src/vue-app)）
   - 基于 Vue 3 + Element Plus 的新 UI 系统
   - 使用组合式 API（Composables）
   - 与传统 UI 共享状态和事件系统

### 核心模块

- **Editor.ts** — 编辑器主入口，负责初始化各层和模块
- **Modules.ts** — 模块管理器，维护编辑器各功能模块的引用
- **EditorData** — 全局编辑器数据，存储当前场景、选中对象等状态
- **editorui**（[src/global/editorui](src/global/editorui.ts)）— UI 层管理器
- **editorRS** / **editorcache** — 资源系统和缓存管理

### 插件是三端包（triple-half）与 VS Code Web

**插件不只是"编辑器的插件"**：目标形态是**三端包**——同一份插件包在三端各有一个入口
（[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) §6.6，D4）：

| 端 | 形态 | 现状 |
|---|---|---|
| **编辑器 Node 端**（宿主） | 宿主侧服务 / 命令，cordis 插件 | **不存在**（插件目前只在构建期装载） |
| **编辑器 Web 端** | 现有贡献点（面板 / 浮层 / Logic / 属性控件 / 桥接方法）；将来走 slots 契约 | **已成型** |
| **游戏项目端**（runtime） | 在游戏产物内运行；**只能依赖引擎 API（feng3d），禁止依赖编辑器 API** | **不存在** |

两条通道**别混为一条**：编辑器 Node 端 ↔ Web 端是**实时通道（WebSocket）**；
编辑器 → 游戏项目端是**产物通道（文件级契约）**——游戏端**不连 WebSocket**，离线运行、只读产物。
**只要插件引入了新的 `__type__`，第三端就不是可选项**（编辑格式 = 运行格式，D1）：两端都要注册对应 Logic。

**VS Code Web**：文件树 / 脚本编辑 / 终端 / Git / 全局搜索交给 **VS Code Web**（D11），
editor Web **不再做文件管理**，专注 3D 场景、属性配置、产物生成入口与插件管理；
内嵌的 `packages/codeeditor` **可废弃**。
两者界面关系（两个并列页面 / 一方嵌入另一方 / 把场景视图做成 VS Code 自定义编辑器）**属未决策项**
（[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) §11 问题 14）。

### packages 工作区（**已按实际更正**，见 [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) §6.10）

> **更正**：本节原先声称"本包使用 npm workspaces 管理子包"，并列出 `cannon` / `cannon-plugin` /
> `themes` / `objectview` 等目录——其中多个**已不存在**，且"workspaces 管理"与**实际不符**：
>
> - 根 `package.json` 的 `workspaces` 是 `["packages/*", "packages/*/examples", "examples"]`，**不递归**，
>   所以 `packages/editor/packages/*` **不在任何 workspace 内**；
> - `packages/editor/package.json` **没有 `workspaces` 字段**；
> - **后果**：这些子包**既不参与 `npm i` 安装，也不参与发布**。
>
> 目录现状（`packages/editor/packages/`）：
>
> | 目录 | 现状 |
> |---|---|
> | `native/` | `NativeFSBase.js`（基于 `fs-extra` 的 Node FS 实现）+ package.json；`main: index.js` 指向**不存在的文件** |
> | `typescript/` | 有源码、**没有 package.json → 不可发布**；全仓无引用 |
> | `codeeditor/` | Monaco 独立窗口（`window.opener` / AMD / DOM），`private: true`；D11 后可废弃 |
> | `editor/` | **空目录**（无 package.json、无入口） |
>
> 它们**是收进 workspace 还是就地删除重写，属未决策项**（[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) §11 问题 9），
> 本文不替它下结论。上文「核心模块」与「架构概览」讲的单例 / 贡献点机制都在
> `packages/editor/src/**`，**与这四个子包无关**。

### 构建配置

- **多入口构建**：index.html（编辑器主界面）、run.html（运行预览）
- **外部依赖**：feng3d 及相关插件通过 CDN 加载，**不打包进 bundle**
- **静态资源**：resource/ 目录在构建时复制到 public/
- **类名保持**：esbuild 配置 `keepNames`，避免类名被压缩修改

## 代码规范（本包补充）

### Vue 组件

- **逻辑抽离**：`.vue` 文件只保留 template，TypeScript 逻辑抽离到同名 `.ts` 文件
- **样式抽离**：CSS 放在 `styles/` 目录独立文件
- **组合式函数**：使用 `useXxx` 命名的 composables 封装逻辑
- **响应式对象**：命名以 `r_` 开头（如 `r_owner`）；不传入函数参数、不导出；仅在需要 `computed`/`watch` 时使用，并在组件或 composable 内部创建

### VSCode 主题

- 所有颜色变量由 `ThemeService` 从 VSCode 主题文件动态加载
- **不在 CSS 中硬编码** VSCode 主题颜色
- 变量名直接对应 VSCode 原始 key（如 `button.background` → `--button-background`）

### TypeScript

- 变量和函数用 camelCase，类和接口用 PascalCase
- 避免 `any`（eslint 规则虽允许，本包规范不建议）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [feng3d-labs/feng3d](https://github.com/feng3d-labs/feng3d) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
