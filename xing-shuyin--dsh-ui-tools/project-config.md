---
trigger: always_on
description: > 给 AI 代理（Claude / Cursor / pi 等）看的项目地图。改动前先读本文件，尤其是
---

# AGENTS.md — dsh-ui-tools

> 给 AI 代理（Claude / Cursor / pi 等）看的项目地图。改动前先读本文件，尤其是
> 「架构要点与坑」一节——里面记录了多个反直觉的设计约束。

## 项目是什么

dsh-ui-tools 是 **DeepSeek Harness（dsh）web** 的插件：在对话页右侧嵌入一个开发者
工具面板，提供 **文件浏览/预览/提及、多终端、Git、后台服务、设置（行内标记/声音）**
五个 tab。样式与功能 1:1 移植自 [pi-web-ui](https://github.com/xing-shuyin/pi-web-ui)
（本仓库内简称 "pi-web-ui"），已发布 npm `dsh-ui-tools`。

- 插件结构：**双半**——宿主端（Node，跑在 dsh 进程内）+ 客户端（React，esbuild 打成单文件）。
- 宿主端 `dsh/index.js` 是包根导出；客户端产物 `client/client.js` 走
  `window.__ModuleLoader__.load({ id, factory })` 协议被 dsh web shell 加载。
- 关键外部依赖：`node-pty`（终端）、`@xterm/xterm` + `@xterm/addon-fit`（客户端渲染）。

## 目录结构

```
dsh-ui-tools/
├── package.json          # 插件清单（dsh.bundle.patch / dsh.client 元数据）
├── cordis.patch.yml      # bundle 层：把插件插入 profile 的 layer 栈
├── dsh/                  # 宿主端（Node ESM，apply(ctx) 插件格式）
│   ├── index.js          # 入口：HTTP 路由 + TerminalManager(node-pty) + git + 提及注入
│   ├── background-servers.js  # 后台服务检测：tools/execute 前后端口 diff
│   └── marker-todo.js    # 通用内联标记系统(发布 inlineMarkers 服务) + todo 接管
├── client/
│   ├── src/              # 客户端 TSX 源码（esbuild 打包）
│   ├── client.js         # 构建产物（git 跟踪、随包发布）
│   └── bundle-body.js    # 中间产物（gitignore）
├── examples/inline-note/ # 第三方内联标记工具官方示例
├── scripts/build-client.mjs
└── shots/                # README 截图
```

## 宿主端（dsh/）

### dsh/index.js — 插件入口

`export function apply(ctx)`，职责：

- **HTTP 路由**：通过 `ctx.inject(['webServer'], cb)` 注册前缀 `/dsh-ui-tools`
  的 handler（`handleRequest`），全部 API 见下。注意：**webServer 是 scoped 服务，
  插件自己的 ctx 用 `ctx.get('webServer')` 拿不到**，必须用 `ctx.inject` 闭包形式
  （index.js 里注释说明了这一点，与 modlens 同款问题）。
- **TerminalManager**（`class TerminalManager`，端口 415 起）：每个 tab 一个
  node-pty PTY；SSE（`/stream`）推流、POST `input|resize|signal|heartbeat`。
  GC：**无 SSE 连接且 60s 无 heartbeat 的终端被杀掉**（15s 扫一次）。
  终端在 dsh 进程内直接 spawn shell，不经过沙箱（README 有安全提示）。
- **Git**：读操作（status/branch/log/diff）`execFile` 直跑；**写操作
  （commit/push/pull/checkout）不在宿主执行，而是开一个可见的终端 tab 跑命令**
  （客户端 GitView 通过 termStore.addTab 加 tab 并切到终端 tab）。
- **文件提及注入**：`ctx.on('agent/pre-step')` 在用户消息进入 agent 前，把
  「每个提及文件一条、只含路径的引用消息」插到用户消息前面（`source.kind='plugin'`
  防止再次富化；`enrichedIds` 防重复，>10000 清空）。**不附加文件内容**——agent 自己
  用 read 工具按需读，省 token。
- **node-pty spawn-helper 权限修复**：启动时检查/修复 `spawn-helper` 可执行位
  （macOS 安装后可能没 +x）。

### dsh/background-servers.js — 后台服务检测

- hook `tools/execute` 的 bash/pwsh（`ctx.on('tools/execute', ..., true)` prepend，
  waterfall 环绕 `await next()`）：执行前快照 LISTENING 端口，完成后 1.5s 再快照，
  diff 出新监听的端口 → 记录 `{ port, pid, since, name? }`（跨会话存活的插件级全局 Map）。
- 每 30s prune：端口 + pid 双匹配，进程退出自动移除。
- kill：`killPidTree`（POSIX `process.kill(-pid)`；Windows `taskkill /F /T`）。
- 与 todo 完全独立，只管「真实运行的服务」。

### dsh/marker-todo.js — 行内标记 + todo 接管

- 解析 assistant 消息正文里的 `[[tool:op:args]]` 标记（`TOKEN_RE`），按注册表分发；
  **反引号/代码块内的标记不执行**（先做等长掩码再匹配）；`[[` 嵌套防御。
- 通用注册表以 Cordis Service **`inlineMarkers`** 发布（`ctx.provide`），第三方
  bundle 用 `inject: ['inlineMarkers']` + `ctx.inlineMarkers.register(name, { applyOp })`
  注册新标记（见 examples/inline-note/）。
- todo 功能：原生 `todo_write` 工具被 `tools.restrict({ deny: ['todo_write'] })` 隐藏；
  `tools/execute` 环绕兜底旧调用；持久化沿用原生 `todo/write` 事件（`data.todos` 原生
  投影 + `data.marker` 内嵌富状态）；只读查询 `markers_list` 工具；系统提示指引
  （order 150）。**写入前必须做无损 JSON 清洗**（Session.append 拒绝 undefined/BigInt
  等，见文件头「序列化约束」）。
- 设置：`createMarkerController(ctx)`（宿主端 `~/.dsh-ui-tools/settings.json`）——
  全局总闸 + 插件级开关，运行时切换即时生效。

## 客户端（client/src/）

- **`index.tsx`** — 客户端入口 `apply(ctx)`：注入 `<style>`（pluginCss + xtermCss，
  卸载时移除）；`initLayoutControl()`；`replaceSessionLogLabel()`；`initThemeWatch()`；
  注册 slots：
  - `details`（priority -1）→ `<Panel>`（右侧面板本体，阴影掉内置 DetailsPanel）
  - `conversation.input.dock` ×2 → MentionStrip（order 30）、SoundWatcher（order 20）
- **`Panel.tsx`** — 面板外壳：5 个 tab（文件/终端/Git/任务/设置）+ 收起/悬浮重开 +
  左缘拖拽调宽（`setDetailsWidth`）。所有视图常驻挂载、CSS 隐藏（保终端回滚/文件树状态）。
- **`store.ts`** — 模块级 observable 仓库（panel tab / terminal tabs / mentions），
  **不是 React state**——因为 slots 按会话独立挂载/卸载。`termStore.addTab` 自动激活新
  tab（Git tab 写操作依赖此行为）；`nextTitle()` 生成 `终端 N`。
- **`api.ts`** — 类型化 fetch 封装 + `rawUrl`/`encodeCwd`。
- **`layout.ts`** — 右侧列宽控制：**shipped layout controller 不可靠**，所以直接用 CSS
  `!important` 覆盖布局 frame 的 inline `grid-template-columns`，用两个 CSS 变量驱动：
  `--ut-sidebar-px`（MutationObserver 从 frame inline style 同步）+ `--ut-details-px`
  （打开=宽度 px，收起=0px，localStorage 持久化）。frame 选择器：唯一带 inline
  `grid-template-columns` 的 div，找不到会重试 60 次×500ms。
- **`theme.ts`** — 观察 `body[data-ds-dark-theme]` 属性增删（MutationObserver），
  给需要 JS 读色的 xterm 配色提供 observable；CSS 层全部走 `--dsw-alias-*` token 无需 JS。
- **视图组件**：
  - `FilesView.tsx` — 面包屑 + 扁平文件列表 + 工作区切换器
  - `FilePreview.tsx` — 预览弹窗（行号/行选择/编辑 Ctrl+S/Markdown 渲染切换/hex/媒体）
  - `TerminalView.tsx` — 快捷命令条 + 终端 tab 条 + xterm（一个 tab 一个实例，隐藏的
    保持挂载）；「切换进终端 tab 且无终端」时自动新建（`active && tabs.length === 0`）
  - `GitView.tsx` + `git-parse.ts` — Git 面板 + 输出解析器
  - `JobsView.tsx` — 后台服务列表（5s 轮询）
  - `SettingsView.tsx` + `SoundSettings.tsx` — marker 开关 / 声音设置
  - `MentionStrip.tsx` — 输入框上方提及条；`SoundWatcher.tsx` — 事件边界播提示音
- **`sounds.ts`** — Web Audio API 合成提示音（无音频文件），设置存 localStorage。
- **`markdown.ts`** — 零依赖迷你 Markdown 渲染器（先 escape 再变换，无裸 HTML）。
- **`shell-labels.ts`** — MutationObserver 把硬编码的 "Session log" 文案改成 "log"
  （locale override 够不到硬编码叶子文本）。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xing-shuyin/dsh-ui-tools](https://github.com/xing-shuyin/dsh-ui-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
