---
trigger: always_on
description: > 面向**消费插件开发者**：如何让你的插件向 better-sidebar 注册新的侧边栏页面（tab）和文件类型预览器。
---

# dsh-better-sidebar 插件接入文档

> 面向**消费插件开发者**：如何让你的插件向 better-sidebar 注册新的侧边栏页面（tab）和文件类型预览器。

better-sidebar 从 v0.4.0 起暴露 `ctx.betterSidebar` 服务（Cordis context 属性），其他插件通过 `registerTab` / `registerFileViewer` 注册扩展点，返回 disposer 由 Cordis fiber 自动管理生命周期（HMR-safe）。

---

## 0. 仓库硬约束（所有本仓库插件必须遵守）

- **禁止修改 DeepSeek Harness (DSH) 源码**：对官方源码 checkout（`~/.dsh/source/current`）零写入——不得改 harness 包、不得把 harness 改动提交到它的分支。
- **挂载只走 `cordis.patch.yml` + profile 机制**（`~/.dsh/profiles/<profile>/`），插件永远作为独立包被 profile 引用，不反向侵入 DSH。
- 需要 harness 没有的能力时，用 DSH **现成的只读/公开 API** 或插件自有路由实现（参考 §7 的 `jobs.output` 事件回放：读会话事件日志而非动注册表）；如果确实做不到，先向用户说明取舍，而不是直接改 DSH。

---

## 1. 服务定位

- **服务名**：`betterSidebar`（即 `ctx.betterSidebar`）
- **发布侧**：better-sidebar 的 client half（`src/client/index.tsx`，通过 `ctx.provide('betterSidebar', service)` 发布）
- **消费侧**：你的插件的 client half（`inject = ['betterSidebar', ...]`，然后 `ctx.betterSidebar.registerTab(...)`）
- **类型合并**：`declare module 'cordis' { interface Context { betterSidebar: BetterSidebarService } }` 由 `dsh-better-sidebar` 包导出；消费插件 `import type {} from 'dsh-better-sidebar'` 即触发类型合并

> ⚠️ **host 半不发布此服务**：`ctx.betterSidebar` 只在 client 侧存在。如果你的插件 host 半需要读 better-sidebar 状态，走 better-sidebar 自己的 HTTP/WS 路由（`/sidebar/api/*`），不走服务。

---

## 2. 消费插件的最小骨架

### 2.1 `package.json`

```jsonc
{
  "name": "my-plugin",
  "peerDependencies": {
    "cordis": "^4.0.0-rc.7",
    "dsh-better-sidebar": "workspace:*"
  },
  "peerDependenciesMeta": {
    "dsh-better-sidebar": { "optional": true }
  }
}
```

- `dsh-better-sidebar` 必须声明为 **peerDependency**（不是 dependency，避免重复实例化）
- 标记 `optional: true` 让你的插件在 better-sidebar 未安装时也能加载（注册代码会因为 `ctx.betterSidebar` 为 undefined 而跳过）

### 2.2 client half 入口

```ts
// my-plugin/src/client/index.ts
import type {} from 'dsh-better-sidebar'  // 触发 ctx.betterSidebar 类型合并

export const inject = ['betterSidebar', 'slots']  // 声明服务依赖

export function apply(ctx: Context): void {
  // 注册一个 sidebar tab
  ctx.effect(() =>
    ctx.betterSidebar.registerTab({
      id: 'my-plugin:db',
      title: () => 'Database',
      icon: <DbIcon />,
      order: 50,
      component: ({ ctx, scope, tab }) => <DbView sessionId={scope.sessionId} />,
    })
  )

  // 注册一个文件预览器
  ctx.effect(() =>
    ctx.betterSidebar.registerFileViewer({
      id: 'my-plugin:csv',
      exts: ['csv'],
      fetchStrategy: 'custom',
      load: async (path, scope) => parseCsv(await fetchCsvBytes(scope, path)),
      component: ({ customData, path }) => <CsvGrid data={customData} path={path} />,
    })
  )
}
```

> ⚠️ **构建期纯度门**：client bundle 禁止 value-import 别的插件代码（`tsdown.config.ts` 的纯度门会挡）。`import type {}` 会被擦除，**不触发门禁**——所以类型可以自由共享，运行时符号不行。所有运行时交互必须走 `ctx.betterSidebar` 的方法调用。

### 2.3 类型导入

```ts
import type { TabDescriptor, FileViewerDescriptor, BetterSidebarService } from 'dsh-better-sidebar'
```

类型定义在 `lib/types/client/service.d.ts`，通过 `package.json` 的 `./client/service`（别名 `./client/api`）exports 子路径暴露。

---

## 3. Tab 注册 API

### 3.1 `TabDescriptor` 完整字段

```ts
interface TabDescriptor {
  /** 唯一 id；也是 SidebarTab.type 的值。建议带包前缀：'my-plugin:db'。 */
  id: string
  /** 标题（i18n 友好：传字符串或返回字符串的函数） */
  title: string | (() => string)
  /** 图标：ReactNode 或 (size: number) => ReactNode */
  icon?: ReactNode | ((size: number) => ReactNode)
  /** + 菜单排序（升序）；默认 100。内置：explorer=10, git=20, subagent=30, terminal=40 */
  order?: number
  /** 从 + 菜单隐藏（editor/diff 用：由其他流程触发打开，不在菜单里） */
  hidden?: boolean
  /** + 菜单禁用判定（如 terminal 配额满）。三参：ctx、会话 scope、当前状态 */
  available?: (ctx: Context, scope: SessionScope, state: SidebarState) => boolean
  /**
   * 单实例语法糖：`single: true` ≡ `dedupeKey: () => id`（打开时聚焦既有
   * 同类型 tab 而非新开）。显式给出 dedupeKey 时优先于 single。
   */
  single?: boolean
  /**
   * 去重键：openTab 时若已存在 dedupeKey 相同的 tab，则聚焦而非新开。
   * 返回 undefined 表示不去重（每次都新开，但同 id 会被 id 安全网聚焦）。
   * 内置策略：explorer/git/subagent 用 single: true；editor 用 tab => tab.path；diff 用 tab => tab.id。
   */
  dedupeKey?: (tab: SidebarTab) => string | undefined
  /**
   * 自定义 tab 创建（minting SidebarTab + 状态 patch）。
   * 返回 null 拒绝创建。terminal 用它生成 terminal:<n> id 并递增 nextTerminal。
   * 省略时用默认 { id, type, title } + seed 里的 path/diff。
   */
  createTab?: (state: SidebarState) => { tab: SidebarTab; patch?: Partial<SidebarState> } | null
  /**
   * 声明式设置（v0.4.1+）：每个注册的 tab 都会在 Side card 设置页获得一行
   * 开关（图标 + 标题 + 类型 id），`settings.toggles` 在其行下追加嵌套开关，
   * 绑定 SidebarPrefs 字段。嵌套开关仅父级启用时显示。
   */
  settings?: {
    toggles?: readonly {
      /** SidebarPrefs 字段名（内置键：'autoOpenSubagent' / 'agentTerminalTools' / 'htmlViewerNoSandbox' / 'htmlViewerDefaultUnsafe' / 'browserNoSandbox' / 'browserInterceptLinks'） */
      key: string
      title: string | (() => string)
      desc?: string | (() => string)
    }[]
  }
  /** 渲染函数 */
  component: (props: TabComponentProps) => ReactNode
}
```

### 3.2 `TabComponentProps`

```ts
interface TabComponentProps {
  ctx: Context                 // client cordis context
  store: SidebarStore          // better-sidebar 的状态 store（可调 reduce 等）
  scope: SessionScope          // { sessionId, cwd? }
  tab: SidebarTab              // 当前 tab 实例（含 id/type/title/path?/diff?）
  visible: boolean             // 是否是当前激活 tab 且面板打开（不可见时暂停轮询等）
  // 以下由内置 tab 使用，外部 tab 可忽略：
  expanded?: string[]          // explorer 的展开目录集
  onToggleDir?: (path: string) => void

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [omdsh-dev/DSH-better-sidebar](https://github.com/omdsh-dev/DSH-better-sidebar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
