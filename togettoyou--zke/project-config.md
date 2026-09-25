---
trigger: always_on
description: 本文件适用于 `web/console/` 及其所有子目录，在仓库根 `AGENTS.md` 的基础上补充前端约束。两者冲突时，根文件的
---

# ZKE Console 前端开发指南

本文件适用于 `web/console/` 及其所有子目录，在仓库根 `AGENTS.md` 的基础上补充前端约束。两者冲突时，根文件的
产品、架构、安全与 Git 纪律优先，本文件负责界面实现。根文件的 Canary 规则同样适用。

## 技术栈与命令

Vite 8 + React 19 + TypeScript 5.9（`strict`），pnpm 11 / Node 24；Tailwind CSS 4（语义变量集中在
`src/styles/theme.css`）、Radix UI、TanStack Query 5 与 Table 8、Zustand 5、Zod 4 + react-hook-form、
Sonner（经 `components/common/toaster.tsx`）、xterm.js；时序图表使用 uPlot。API 类型由 `openapi-typescript`
从 `api/openapi/zke-server.v1.yaml` 生成。

```bash
pnpm dev            # 开发服务器
pnpm typecheck      # tsc -b
pnpm lint           # eslint
pnpm format         # prettier --write
pnpm build          # tsc -b && vite build
pnpm gen:api        # 重新生成 src/api/schema.d.ts
```

`src/api/schema.d.ts` 是生成文件，不得手工编辑；OpenAPI 契约变更后运行 `pnpm gen:api`。

## 目录分层

```
src/
  api/        openapi-fetch 客户端、错误映射、SSE、按领域划分的 query hooks
  apps/       桌面应用。每个应用一个目录，registry.ts 是唯一的应用清单
  auth/       登录、首次初始化、会话与权限能力
  components/ui/      无样式原语的样式化封装；common/ 跨应用复合组件；brand/ 品牌标记
  desktop/    桌面外壳：窗口、Dock、顶栏、启动器、持久化
  scope/      租户/项目作用域选择器与 store
  lib/        无 UI 依赖的纯工具
  styles/     theme.css，全部设计变量的唯一来源
```

依赖方向：`apps` → `components` → `ui`/`lib`。`components/ui` 不得引用 `apps`。`api/queries` 只在需要暂停轮询
时引用 `desktop/window-visibility`，除此之外不得依赖外壳。新增原语放 `components/ui/`，跨应用复合组件放
`components/common/`，只有一个应用用到的留在该应用目录内。

## 设计系统：不可协商

**颜色只允许语义 token。** 禁止 Tailwind 调色板（`bg-blue-500`）、字面色值，以及未在 `theme.css` 中定义的
token。完整集合：

- 表面 `surface`、`surface-muted`、`surface-raised`、`surface-overlay`
- 文字 `foreground`、`muted-foreground`、`subtle-foreground`
- 描边 `border`、`border-strong`
- 主色 `primary`、`primary-hover`、`primary-foreground`、`primary-surface`
- 语义 `success`、`warning`、`danger`、`info`、`neutral`，各带 `-surface` 变体
- 焦点 `ring`；代码 `code-key`、`code-string`、`code-literal`、`code-comment`、`code-punctuation`、`code-meta`；
  桌面 `desktop-from`、`desktop-to`

不存在 `background`、`muted`、`card`、`accent`、`destructive`、`warning-foreground` 等 shadcn 习惯名。Tailwind
对未定义 token 不报错，只静默不生成类——`bg-background` 的元素是完全透明的。新增 token 必须同时加进
`theme.css` 的 `:root`、`[data-theme="dark"]` 和 `@theme inline` 三处。两个例外已在 `theme.css` 写明理由：
应用图标面上的字形恒为 `text-white`，对话框遮罩为 `bg-black/35`。**深浅色是一等公民**，任何新界面都必须在两套
主题下成立。

**圆角**只用 `rounded-inline`(4，行内文字按钮焦点光晕) / `rounded-control`(7，按钮、输入框、下拉项) /
`rounded-panel`(10，卡片、表格容器、菜单、弹层) / `rounded-window`(12，窗口与对话框) / `rounded-full`；不用
`rounded-sm|md|lg|xl|2xl` 和裸 `rounded`。应用图标瓦片不在此刻度上：圆角是边长的 31.25%，使同一图标在启动器、
Dock 和品牌位是同一形状。

**字号**只用 `text-[11px]`（外壳次级说明）/ `text-xs`(12，表头、提示、元信息) / `text-[13px]`（正文默认）/
`text-sm`(14，区块标题) / `text-[15px]`（对话框标题）/ `text-[22px]`（页面标题与概览大数字）。不引入新的中间值。

**高度**用 `shadow-e1|e2|e3` 与 `shadow-window|window-focused`，不用 Tailwind 默认阴影。**卡片不叠阴影**：窗口
本身已有高度，窗口内的卡片与表格容器一律靠 `border` + `bg-surface` 划分。

**焦点**一律 `zke-focus`（复合控件 `zke-focus-within`），它画紧贴边框的 3px 光晕。不要写 `outline-none` 而不补
焦点样式，也不要用默认的偏移 outline。

**动效**统一定义在 `theme.css`，组件只挂类名：`zke-overlay-motion`（遮罩）、`zke-dialog-motion`（对话框本体）、
`zke-pop-motion`（菜单、Select、Popover、Tooltip，按 Radix `data-side` 展开）、`zke-window-motion`、`zke-rise`、
`zke-interacting`。新的瞬态浮层必须挂对应类名，退场动画依赖 Radix 的 `data-state="closed"`，删掉动画等于删掉
退场。曲线只有两条：入场 `--ease-lift`，大位移 `--ease-reveal`；离场一律 `linear`。本项目**刻意不实现**
`prefers-reduced-motion`，原因写在 `theme.css` 末尾，要改这个决定先读那段注释。

## 尺寸与输入方式

Console 是一个桌面外壳，但它跑在浏览器里，所以同一份代码要同时成立于 1440px 的显示器和 390px 的手机。下面的
规则没有一条是在「判断设备」。

**问窗口有多宽，不问屏幕有多宽。** 窗口内部的响应式一律用容器查询（`@md:`、`@2xl:`、`@max-2xl:`），不用视口断点
（`sm:`、`md:`、`lg:`）。桌面隐喻下这两个数字只是偶尔相等：宽屏上被拖窄到 420px 的窗口，用视口断点会按 2560px
来排版。容器由外壳声明，应用直接用即可：

- `desktop/Window.tsx` 的内容层是 `@container`——不走 `AppShell` 的应用（AIOps）问的就是它；
- `apps/AppShell.tsx` 的根与工作区各是一个 `@container`——应用里写的 `@md:` 问的是工作区的实际宽度，也就是窗口
  减去导航与内边距之后剩下的那点；
- `components/ui/dialog.tsx` 的 `DialogContent` 是 `@container`——对话框被 portal 到 body，没有祖先容器，容器查询
  会永远不成立，所以它自己声明一个。

栅格换列的阈值按「一列还能放下一个可用字段」定：两列 `@md`(448)、三列 `@2xl`(672)、四列与五列 `@3xl`(768)。
`col-span` 必须跟随它所在栅格的阈值，不要照抄前缀。

**阈值有上限，而且比直觉低。** 容器查询问的是工作区，不是窗口，更不是屏幕：容器服务默认 1060px 的窗口减去
160px 导轨、内边距和边框，工作区只有 866px。任何高于 `@3xl`(768) 的阈值在默认窗口里都不会成立——`@4xl` 是
896，差 30px，四列会静默塌成两列，而在桌面上这是一次肉眼可见的回归。定阈值时先算目标应用默认窗口的工作区宽度，
再挑不超过它的那一档；`@4xl` 及以上只留给本来就铺满窗口的布局（例如 AIOps 的导轨轨道，它的容器是整个窗口）。

例外只有两处，都不在窗口里：`auth/LoginPage.tsx` 与 `desktop/TopBar.tsx` 直接铺在视口上，用视口断点是对的。

**侧边栏一律可收缩，收缩规则只有一份。** 带侧边栏的两个外壳——`apps/AppShell.tsx` 的应用导航与 AIOps 的会话
列表——共用 `apps/sidebar.ts` 的 `SIDEBAR_COLUMN_MIN_WIDTH`(672) 与 `useNarrowSurface()`，行为必须一致：

- 展开是一列（`AppShell` 160px，AIOps 204/256px），收起是一条 52px 的图标条，标签退到 `HintTooltip`；折叠
  开关挂在列表**下方**、一条分隔线之后，和图标同列——放在上方会多出一条空带，把第一项顶得和右侧工具栏行错位；
- 载体宽度低于阈值时，展开态脱离文档流浮在工作区之上（`@max-2xl:absolute`，且必须换成不透明底色），工作区始终
  拿到全宽——绝对定位的子元素不是 grid item，栅格要相应减掉那一条轨，否则内容会被挤进 0px 轨；
- 用户没表过态时跟随载体宽度（`railChoice ?? narrow`），显式切换后在任何宽度都保持；在浮层态下选中一项时释放
  这个覆盖（置回 `null`）而不是新设一个，这样窗口变宽还能把列还回来；
- 布局归 CSS 容器查询，`useNarrowSurface()` 只用来驱动「选中后收起」这类行为，两者读同一个数。

**问指针粗细，不问是不是手机。** `theme.css` 定义了两个变体：`coarse:`（`pointer: coarse`，手指按不准）与
`hoverless:`（`hover: none`，没有「悬停」这个中间态）。触摸笔记本是 coarse 且能悬停，带触控板的平板是 fine 且能
悬停——只有问输入方式才有确定答案。

- 点击目标：`components/ui` 的控件都带 `zke-control`（方形的再带 `zke-control-square`），粗指针下最小 40px，细指针
  下完全不生效。行内的文字型点击点用 `zke-touch-pad`，它不改变绘制尺寸，只在背后挂一块 44px 的响应区；
- 悬停才出现的东西，在 `hover: none` 上等于不存在。任何 `group-hover:opacity-100` 都要补 `hoverless:opacity-100`，
  或者说明为什么这个入口在触摸设备上可以没有。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [togettoyou/zke](https://github.com/togettoyou/zke) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
