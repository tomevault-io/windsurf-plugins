---
trigger: always_on
description: 给 Codex 在本仓库工作时的说明。`README.md`(英文)与 `README.zh-CN.md`(中文)是面向用户的权威文档。
---

# AGENTS.md

给 Codex 在本仓库工作时的说明。`README.md`(英文)与 `README.zh-CN.md`(中文)是面向用户的权威文档。
`packages/app/AGENTS.md` 和 `packages/desktop/AGENTS.md` 是包级规则,必须遵守。

## 这是什么

Yoma 是一个面向**嵌入式调试**的 agent 平台,一棵树上两半:

- **内核**(`packages/{ai,agent}` 两个上游包 + `packages/kernel/src/host`)—— agent 循环、会话树、
  压缩、技能,以及嵌入式应用层(工具链解析 / 示例语料 / 引擎调用)。嵌入式工具组(烧录 / 日志 / gdb /
  网表 / 数据手册 / STM32 配置 / 逻辑分析仪 / 示波器)2026-09-10 **归零**:旧实现搬到
  `packages/kernel/attic/`(不编译、不跑),按新内核的工具接口一个个重写 —— 2026-09-11 起按样板
  `host/tools/<名字>/{contract.ts,session.ts}` 逐个重写,首个是 flash,2026-09-12 起有 grep / find / ls / powershell,2026-09-14 起有 log(串口 / TCP / 命令三种源)。示波器 2026-09-16 恢复为
  `host/tools/scope/` + `host/domain/scope/`,2026-09-17 分出 `ScopeDriver` 接口 + 驱动注册表(siglent 一族,USB 与 LAN;demo 按环境变量),带只读历史波形面板;Mac 首轮真机检查通过(重构后未再上真机),Windows 与故障恢复稳定性仍待验证,见 `docs/scope-usb.md` 与 `docs/scope-drivers.md`。
  例程库仍停在仓库外 `../yoma-parked/`。
- **桌面端**(`packages/{desktop,app,kernel,ui,session-ui,util,bench}`)——
  Electron 外壳 + SolidJS UI,fork 自 opencode 的前端;`bench` 是无人值守调试台。

**2026-08 之前这是两个仓库**(`yoma` 和 `yoma-desktop`,兄弟目录 + alias 接缝)。
合并的决定性理由是**它们从来不独立发布**:打包时 esbuild 把内核源码整个 inline 进
`out/main/kernel.js`,用户装的 app 里没有"内核这个包",只有一个把两边融在一起的产物。
仓库该按发布节奏切分,而这两半的发布节奏不是相近 —— 是同一个。

分开时代付出的代价(现在都没了):路径映射要维护 4 份、`bun use-yoma` 切检出、
"半切"(app 跑新代码而 typecheck 验旧检出,两边全绿却说的不是同一件事)、
以及**跨仓库的静默断裂** —— 一天之内撞过三次,其中"凭据路径 + 格式变了"那次
类型系统根本抓不到,表现是用户配了 key 而内核静默读不到。

## 内核接缝:一个包、几道门

内核就是本仓的 workspace 包 `@yoma-desktop/kernel`,裸说明符靠 **node_modules 里的软链 +
它自己的 `exports`** 解析 —— typecheck(tsgo)、tsx、vitest、esbuild / electron-vite 全走这一条。
**别名表没有了**(2026-09-10 删:`kernel-alias.ts`、四处 `paths`、各处 `resolve.alias`;
守门的活交给 `packages/kernel/src/host/boundary.test.ts`)。

四个盒子(`boundary.test.ts` 的说法):**餐厅** = app / session-ui / ui / util / desktop,只认**菜单**
(kernel 的门 `.`,浏览器安全);**厨房** = kernel 的门 `./host` + bench;**工具间** =
`kernel/src/host/domain/` 与 `host/tools/<名字>/{contract.ts,session.ts}`(2026-09-11 起有住户:flash);
**发动机** = `packages/{agent,ai,chord,telemetry}`(哈希锁定)。

门就是 `packages/kernel/package.json` 的 `exports`,七道(外加 `./package.json`):

| 门 | 谁用 |
|---|---|
| `.`(`src/index.ts`) | 餐厅:视图模型 / 协议 / 客户端,**浏览器安全** |
| `./host`(`src/host/index.ts`) | 厨房大门:desktop 的 `kernel-entry.ts` 与 bench |
| `./host/datasheet-server`、`./host/models`、`./host/toolchain-schema` | 三道**叶子**门:desktop main 的手册库页、bench 的模型目录与信箱工具链清单 —— main 走大门等于把整个 host inline 进 `out/main/index.js` |
| `./tools/*/contract`(`src/host/tools/*/contract.ts`) | **契约门**:餐厅的工具卡片只从这里拿一个工具的名字 / 参数 / 结果格式 / 副标题函数,拿不到 `session.ts`。2026-09-11 起有住户了(flash) |
| `./tools/contracts`(`src/host/tools/contracts.ts`) | **契约总表**:餐厅按工具名找契约(只 import 各 `contract.ts`);装配在 `host/tools/index.ts`,那是厨房 |

- 内核必须被 electron-vite **inline**,所以它得留在 `packages/desktop` 的 **devDependencies** 里
  (`externalizeDeps` 只外部化 `dependencies`)。它只发 raw TypeScript(`exports` 指向 `src/*.ts`,
  内部大量 `./x.ts` 后缀说明符),外部化后 Node 的 strip-only 加载器报
  `ERR_UNSUPPORTED_NODE_MODULES_TYPE_STRIPPING`,**无 flag 可关**;`host/domain` 里还有约 9 处 TS
  构造器参数属性(加上 `src/client.ts` 的 `KernelError`)会直接 `ERR_UNSUPPORTED_TYPESCRIPT_SYNTAX`。
  inline 时这两样一起消失 —— 也正因为这些参数属性,`tsconfig.yoma.json` 的
  `erasableSyntaxOnly: false` 是**承重的**,别顺手收紧。
- 新开一道深引用 = 改 `exports`(从前是改四份别名表)。`boundary.test.ts` 钉住五条:菜单里没有 Node;
  工具间不反调会话间(`host/domain` 往外只拿 `host/models.ts`、`host/datasheet-server.ts`);餐厅只许走
  `@yoma-desktop/kernel`、`@yoma-desktop/kernel/tools/<名字>/contract` 或 `@yoma-desktop/kernel/tools/contracts`;
  desktop 的 main 只有 `kernel-entry.ts` 能走 `./host`;契约文件按白名单只许 `typebox` 与工具间内部的相对路径
  (不含 `session.ts`),且每个工具目录都得有 `contract.ts`。

## 仓库结构

npm workspace,`packages/` 下 11 个包 —— 四个 pi 上游包(`ai` / `agent` / `chord` / `telemetry`,包名
保留 `@earendil-works/*`,由根 `upstream-lock.json` 逐文件哈希锁定,**源码一个字都不改**,见 `UPSTREAM.md`)
与七个桌面端包。嵌入式应用层**不再是单独的包**:2026-09-10 `@yoma/coding-agent` 并进 `kernel`
(领域代码进 `src/host/domain/`,系统提示词 / 资源发现 / 模型目录 / 数据手册地址进 `src/host/`,
用例进 `packages/kernel/test/`,归零的工具实现进 `packages/kernel/attic/`)。

桌面端这 7 个:

| 包 | 名字 | 职责 |
|---|---|---|
| `desktop` | `@yoma-desktop/desktop` | Electron 外壳:main/preload/renderer、内核进程、打包、自动更新 |
| `app` | `@yoma-desktop/app` | SolidJS UI —— 一个**库**,两个宿主(web + desktop);页面、路由、状态、i18n |
| `kernel` | `@yoma-desktop/kernel` | **内核接缝**:浏览器安全的视图模型/协议/客户端 + Node 侧 host |
| `ui` | `@yoma-desktop/ui` | 领域无关的基础组件(Kobalte)、OKLCH 主题引擎、图标 |
| `session-ui` | `@yoma-desktop/session-ui` | transcript 渲染:消息、工具卡片、流式 markdown、Pierre diff |
| `util` | `@yoma-desktop/util` | 纯函数小工具 |
| `bench` | `@yoma-desktop/bench` | **无人值守调试台**:job 交给内核跑到底,判据自验,产出分支与报告 |

分层单向:`ui`(叶) → `session-ui` → `app` → `desktop`;`kernel` 被 `app`、`desktop`
和 `bench` 消费(`bench` 是 host 的**第二个宿主**,不经 Electron)。

`packages/kernel` 的门(见"内核接缝")必须守住:

- `.`(`src/index.ts`)—— **浏览器安全**,不 import `./host`、不 import `node:*`、不 import 发动机。
  视图模型不解释任何工具的结果:`ToolState` 的 `metadata` 是 `Record<string, unknown>`,
  界面对所有工具统一走 `GenericTool` 万能卡(2026-09-10 卡片归零,专用卡待重写工具时按名注册)。
- `./host`(`src/host/`)—— 只跑在 utilityProcess(与 bench 的子进程)里,碰内核、碰文件系统。
- `host/domain/`(工具间)—— 嵌入式领域代码,往外只拿 `host/models.ts` 与 `host/datasheet-server.ts`,
  碰不到 session-manager / projector / protocol。

## 命令

| 命令 | 作用 |
|---|---|
| `npm run dev:desktop` | 开发模式(renderer 有 HMR;**内核进程没有**) |
| `npm run build:desktop` | 生产构建 → `packages/desktop/out/` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yoma-embedded/yoma](https://github.com/yoma-embedded/yoma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
