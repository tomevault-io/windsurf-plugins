---
trigger: always_on
description: 面向**渲染层**（`src/renderer/src/**`）以及它依赖的 `src/shared/**`、`src/preload/**`。
---

# NextCoWork 前端工程约定

面向**渲染层**（`src/renderer/src/**`）以及它依赖的 `src/shared/**`、`src/preload/**`。
主进程只在跨边界的地方出现。规则按「违反了会出什么事」来写——判断不了的时候，
读被改文件的头注释，那里通常已经写了理由。

---

## 1. 依赖方向：单向，不可绕

```
components/views/shell/settings  →  stores  →  services  →  preload 桥  →  main
                       ↘             ↘           ↘
                            shared/**（纯类型 + 纯函数，谁都能引）
```

硬规则：

1. **只有 `services/**` 能出现频道字符串。** 组件、store、i18n 一律调 `services/*` 的具名函数。
   频道白名单在 `shared/ipc/contract.ts`，preload 运行时再校验一次（`src/preload/index.ts:38`）。
2. **渲染层不 import `src/main/**`，不 import `electron`，不 import node 内置模块。**
   要共享的类型/纯函数放 `src/shared/**`。
3. **`services/**` 不引 store，不引组件。** 它只做「拆 IpcResult 信封 + 返回值/抛 `AgentErrorException`」，
   见 `services/ipc.ts`。需要自己处理失败分支时用 `tryInvoke`。
4. **`on()` 的返回值必须进 `useEffect` 的 cleanup。** 漏一个，HMR 每次热更叠一层监听器，
   表现为一次状态更新触发 N 次重渲染，只在 dev 出现，极难定位（`App.tsx:139` 有反面教材注释）。
5. 每个新增 IPC 调用先在 `shared/ipc/contract.ts` 登记，再在对应的 `services/<域>.ts` 里包一个函数。
   **没有 `services/<域>.ts` 就新建一个**，不要在 store 里直接 `invoke`。

已知违规（碰到那块代码时顺手修，不要专门开一轮重构）：

- `views/chat/ChatView.tsx:17` 从 `main/kernel/tool/builtin/todo` 取 `latestTodosFrom` / `TodoItem`
  —— 应下沉到 `shared/`。
- `stores/plugins.ts`、`stores/documents.ts:220`、`views/extensions/plugins/PluginConfiguration.tsx`、
  `settings/pages/import/ImportPage.tsx:82` 直接写了频道字符串 —— 缺一个 `services/plugins.ts`。

## 2. 目录与命名

| 位置 | 放什么 |
|---|---|
| `components/ui/**` | 无领域知识的原子控件（Button / Dialog / Menu / Toast…）。**新控件先在这里找**，仓库已有 19 个。 |
| `components/**` | 跨视图复用、但带一点领域的东西（markdown、brand 图标、UpdateBanner）。 |
| `shell/**` | 窗口外壳：Tab 条、侧边栏、Dock、状态栏、命令面板、快捷键。 |
| `views/<域>/**` | 一个功能域的全部界面。入口组件叫 `XxxView` / `XxxFeature`。 |
| `settings/**` | 设置浮层；每页收 `SettingsPageProps`（`settings/props.ts`）。 |
| `stores/**` | zustand store，导出 `useXxxStore`。 |
| `services/**` | IPC 封装，一个域一个文件。 |
| `i18n/**` | 全部用户可见文案。 |
| `theme/**`、`styles/theme.css` | token、外观、动效档位。 |

- 组件文件 **PascalCase.tsx**，一个文件一个主组件（辅助小组件可以同文件，别导出）。
- 纯逻辑文件 **kebab-case.ts**（`thread-content.ts`、`dock-layout.ts`、`tab-menu.ts`），Hook 文件 **useXxx.ts**。
- **全仓库只有 `App.tsx` 用 default export**，其余一律具名导出。
- 视图的**懒加载边界只有一处**：`views/registry.tsx`。别在组件自己那层 `lazy`
  （聊天热路径上每条消息过一次 Suspense），也别忘了跟传递依赖——静态 import 一个重视图等于把它的
  依赖拽回主 bundle，该文件头注释记了这次翻车。

## 3. TypeScript

- `strict` + `noUncheckedIndexedAccess` + `noImplicitOverride` + `noUnusedLocals/Parameters`（`tsconfig.web.json`）。
- **渲染层当前 `any` 出现 0 次、`eslint-disable` 出现 0 次。保持这个数字。**
  类型难写时用 `unknown` + 收窄，或把类型补进 `shared/domain/**`。
- 导出的函数、组件**写显式返回类型**（组件用 `React.ReactNode` 或 `React.JSX.Element`，两种都在用，跟邻居一致）。
- 判空写**显式比较**：`x !== null` / `x !== undefined` / `flag === true`，不用真值性。
  理由是这个代码库里 `''`、`0`、`false` 全是有意义的值。
- 可选参数往 IPC 送的时候用 `...(x === undefined ? {} : { x })`（见 `services/sessions.ts:18`），
  不要送 explicit `undefined`。
- 类型 import 用 `import type`，和值 import 分开写或用内联 `type` 修饰符。
- 穷尽性检查靠 `switch` + 联合类型，必要时加一张 `Record<Kind, true>` 的对照表
  （`views/registry.tsx:186`）——漏一种 kind 编译期就挂。

## 4. 代码格式

仓库**没有 Prettier，ESLint 也没有任何风格规则**，所以格式靠约定：

- 单引号、**不写分号**、2 空格缩进、尾随逗号不加。这是绝大多数文件的写法，**新文件按这个写**。
- 少数文件（`views/chat/Composer.tsx`、`views/registry.tsx`、`i18n/index.tsx`、`settings/pages/model/*`、
  `shell/AppShell.tsx` 等约 20 个）被格式化工具处理成了双引号 + 分号。
  **改这些文件时跟随该文件的局部风格，不要顺手整篇改格式** —— 那会把 diff 淹掉。
- 不要引入 Prettier / 改 ESLint 风格规则，除非用户明确要求。

## 5. React 组件

- 函数组件 + Hooks，props 直接写内联对象类型；props 超过 ~8 个或要被 `ComponentProps<typeof X>` 复用时
  才提 `interface`。
- props 上**逐个写用途注释**，尤其是「为什么存在」（`views/chat/ChatView.tsx:59` 的 `readOnly` 是范例）。
- **不做防御式 UI**：后端接不了的操作就别画控件，画出来的每个控件都是一次会失败的承诺。
- 组件里禁止裸文案（见 §6）、禁止裸颜色（见 §7）。
- `key` 的选择是语义决定：换会话必须重建 per-session 订阅 → `key={chatKey(tab)}`；
  换文件必须重建编辑器 → `key={workspaceId}:{path}`。**别为了「消掉警告」随便挂 index。**
- 新组件文件**控制在 400 行以内**。仓库里有 26 个 tsx 超过 400 行、最大 2291 行（`Composer.tsx`），
  那是历史包袱，不是范本。拆的办法是把**纯逻辑抽成同目录的 `.ts`**（见 §9），不是拆成一堆传 20 个 prop 的子组件。

## 6. i18n（硬规则，最常被违反）

1. **一切用户可见文案走 `src/renderer/src/i18n/`**：按钮、标题、空态、占位符、tooltip、
   无障碍标签、状态文字、界面上的错误、带插值的句子，全部包含在内。
   JSX/TSX 里**不准出现中文或英文 UI 字符串**。
2. key 是**稳定的产品概念**，不是翻译过的句子；命名空间用 `域.子域.名`（`chat.tool.failedStatus`）。
   新增 key **必须同时补 `zh-CN` 和 `en-US`**，`i18n/index.test.ts` 会校验两边键一致。
3. **新增一个域就新建 `i18n/<域>.ts`**（照 `git.ts` / `ssh.ts` / `usage.ts` 的样子），
   导出 `xxxZh` / `xxxEn` 再在 `index.tsx` 里 spread。不要继续把 key 往 `index.tsx` 那三千行里堆。
   带参数的文案在独立文件里要显式标 `type Params = Record<string, string | number>`，否则整张表类型不匹配。
4. 组件里用 `useI18n().t(...)`；**store / service 这类非组件代码用 `translate()`**
   （同一张表、同一个 locale，见 `i18n/index.tsx` 里那段说明）。在组件里用 `translate()` 会导致切语言不重渲。
5. **不翻译领域值**：模型名、供应商名、工具名、文件名、协议标识、用户内容。只翻它们周围的话。
6. 插件文案只能注册在 `plugin.<pluginId>.` 前缀下（`i18n/plugin-messages.ts`），不得覆盖内置 key。
7. 新增 locale 时要同时改：`SUPPORTED_LOCALES`、两份 catalog、设置项校验、语言选择器，然后跑 typecheck + test。

## 7. 样式与主题

- Tailwind v4，token 定义在 `src/renderer/src/styles/theme.css` 的 `@theme` 里
  （`--color-canvas` → `bg-canvas` / `text-canvas`）。
- **JSX 里不准出现十六进制颜色、不准用 Tailwind 调色板类**（`bg-gray-800` 之类）。
  当前只有 `views/chat/Thread.tsx` 一处例外。要新颜色就先加 token，并在 `theme.css` 里写清楚它的来历。
- `theme.css` 的数值是**从参考实现截图上量出来的**，注释里标了量法和可信度。改数值要么给出新的量测，
  要么在注释里标明「推的」。深浅两套主题的规律不同（chrome 的方向是相反的），别照一个主题的巧合写规则。
- 条件类名一律用 `cn()`（`lib/cn.ts`，twMerge 冲突消解，后写的赢），组件因此能被外部 `className` 覆盖。
- 半透明叠色（如 `bg-accent/10`）优于新增第 N 个 token —— 换色器换 accent 时它自己就跟着走。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [239573049/NextCoWork](https://github.com/239573049/NextCoWork) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
