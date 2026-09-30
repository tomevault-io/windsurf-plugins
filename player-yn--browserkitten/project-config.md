---
trigger: always_on
description: 产品分层见根 [AGENTS.md](../../AGENTS.md)。Chrome 运输见 [../AGENTS.md](../AGENTS.md)。
---

# src/agent — Session Workspace Runtime

产品分层见根 [AGENTS.md](../../AGENTS.md)。Chrome 运输见 [../AGENTS.md](../AGENTS.md)。

本目录是 **offscreen 里跑的产品 runtime**。侧栏只 `import` 设置、标签、workspace 客户端；**不**在侧栏跑 `ToolLoopAgent`。

入口：`vnext/service/sessionWorkspaceService.js`。每条用户消息：`vnext/sessionWorkspace/sendMessage.js` → AI SDK 7 `ToolLoopAgent`（`sessionAgent.js`，`toolChoice=auto`）。没有手写多步 tool loop，没有 chat/run 模式分裂。

```text
sidepanel workspaceRpc('sendMessage')
  → SessionWorkspaceService.sendMessage
  → sendMessage(store, input)
  → createSessionTools + runSessionToolLoopAgent
```

没有 barrel / `index.js` re-export 层。每个 import 都写具体文件路径：`offscreen/runtime.js` → `vnext/service/sessionWorkspaceService.js` → `vnext/runSession.product.js` → `vnext/sessionWorkspace/index.js`。新增模块不要再建 barrel。

## 目录

| 路径 | 职责 |
|------|------|
| `vnext/service/` | `SessionWorkspaceService`：RPC 门面、abort 注册、把 host 回调接到 SW |
| `vnext/sessionWorkspace/` | 领域：store、group、artifact、FS、prompt、tools、office、sendMessage |
| `vnext/primitives/` | `acquire` / `run` 宿主原语（工具层调用，不是模型直接 API） |
| `vnext/adapters/` | QuickJS `codeRuntime`、sandboxClient、`vendor/ai-sdk-loader.mjs` |
| `vnext/host/` | `workspaceRpc` 客户端、`userStop`、`rpcError`、`browserSysHost`（SW `workspace_sys`）、`tabLease`、`taskScheduler` |
| `vnext/skills/` | 打包 playbook（`SKILL.md` + `skillSource.js`）。**不是工具** |
| `llm.js` `provider.js` `modelCatalog.js` | BYOK OpenAI-compatible HTTPS；每轮 `resolveLanguageModel` |
| `webAcquireSettings.js` | `pagewand_web_acquire` |
| `skills.js` | 用户固化 skill（`pagewand_user_skills`），侧栏用；与 `vnext/skills` 不同 |
| `artifacts.js` `draftStore.js` `state.js` `trajectory.js` `documentRender.js` `render/` | 旧表面。产品交付物走 `sessionWorkspace/artifacts.js`。改功能前先确认调用方 |

## 领域与 store

`DurableSessionWorkspaceStore`（`durableStore.js`）继承内存 `SessionWorkspaceStore`，产品必须 durable。集合：

`sessions` · `groups` · `groupMembers` · `items` · `sessionBindings` · `artifacts` · `fsNodes` · `meta`；blobs 在 OPFS。`executions` / `leases` 是回合书，崩溃后作废。可选 durable task 存在 `meta` 键 `task:{taskId}`（进 IDB 快照）；`ensureTaskForMessage` 只给测试，产品 `sendMessage` 不调用它。

硬边界（`tools.js` 文件头）：**SelectionGroup / WebItem / sessionBindings 不能经工具改**。只有 UI RPC（`createGroup`、`bindGroups`、`syncTabSelection`、clipboard pin…）可以。Guest `run(code)` 没有 store、没有 `chrome.*`。浏览器机器经 **`sys`**（`browserSys.js` → SW `workspace_sys`）：

| 调用 | 作用 |
|------|------|
| `sys.help()` / `inspect view=sys` | ABI 目录（`pawwork-sys-v1`） |
| `sys.capabilities()` | 实时探测用户脚本、debugger、截图、下载可用性与大小限制 |
| `sys.tabs.list` / `current` / `frames` | 标签与 frame（进程表） |
| `sys.tabs.open` / `navigate` / `reload` / `close` / `focus` | 进程控制（已有 `tabs` 权限） |
| `sys.eval({ world, code, tabId, frameId })` | `code` 是 async 函数体。`MAIN` = 页面 JS 堆；`USER` = 自有世界 + DOM |
| `sys.fetch({ as:'page'\|'extension', url, tabId, init })` | 两块网卡：页面身份（MAIN fetch，cookies+Referer；用户要的登录态/验证码/本机 IP 链接默认这条）vs 扩展身份（`credentials:'omit'`，无 cookie）。省略 `as` 时宿主仍当 extension |
| `sys.cdp` | CDP 管道：`{ method, params }` 自动 attach；`action: attach\|detach\|events\|targets` |
| `sys.download` / `sys.screenshot` | `chrome.downloads.download`：本 profile cookie jar + 本机 IP，无标签 Referer（不是 extension fetch）。登录态/Referer/验证码优先 `as:'page'`。截图为视口合成 |

`eval` / page `fetch` 走 `chrome.userScripts.execute`。`sys.cdp` 走 `chrome.debugger`（一条管道，不是网络/PDF 产品）。DevTools 已挂上时会 `CDP_BUSY`。返回值必须能 JSON 序列化。`eval` / page `fetch` / `cdp` 只允许 http(s) 可注入页；扩展预览页返回 `NEED_PAGE`（MAIN world 不能碰到 `chrome.*`）。除 `tabs.current` 外，未带 `tabId`/`defaultTabId` 也是 `NEED_PAGE`。`eval` / page `fetch` / `tabs.navigate|reload|focus` / `cdp` 会先占 SW tab 租约，他 session 占用同一 tab → `TAB_LEASED`。`SYS_ABORTED` / `SYS_TIMEOUT` 表示等待结束，不表示副作用已撤销。可见标签截图失败为 `TAB_NOT_VISIBLE` / `TARGET_CHANGED`。`targetId` 会先经 `getTargets` 校验 URL。

`sys.fetch` / `sys.screenshot` 可传 `saveTo: '/scratch/…' | '/artifacts/…'`：宿主写入 guest FS，返回文件回执；`run` 登记交付物。调用携带身份与截止时间，错误保留 `code`。停止可中止扩展 fetch，但已派发的页面/CDP 副作用可能已发生，超时后需查证状态。

Artifact 具有 `revision`；`updateArtifact` 可携带 `expectedRevision`，不同内容的过期写入返回 `ARTIFACT_CONFLICT`。三种画布（sheet / doc / web）与常用 office 写入路径已接入；raw guest 写主文件推进 revision，但仍不是带读版本的条件写入。

模型可见面：没有单独的 `sys` 工具。ISA 写在 `run` 的 `code` 字段（`SYS_MODEL_HINT`）；`run.description` 只指向该字段与 `inspect view=sys`。`inspect.view` enum 含 `sys`、每轮 world 有 `browserSys=pawwork-sys-v1`。完整目录仍是 `inspect view=sys` 或 guest `sys.help()`。

Guest FS（`fs.js`）：

| 访客路径 | 宿主映射 | 权限 |
|----------|----------|------|
| `/context` | `/session/{id}/context` | 只读 |
| `/artifacts` | `/session/{id}/artifacts` | 持久读写 |
| `/scratch` | `/tmp/{id}/{executionId}` | 本轮；无 execution 则拒绝 |

## 一回合

`sendMessage.js`：写入 user message → `beginExecution` → 组世界索引（bound groups/items、artifact 概览、focusPage）→ `buildSessionAgentInstructions` + 本轮 world block → `createSessionTools` → `inventoryFromSession`（**瞄准目标，不隐藏工具**）→ `runSessionToolLoopAgent`。

普通 `sendMessage` **不创建、不附着** durable task（`sessionWorkspaceService.sendMessage`：只有 `taskRun && taskId` 才绑定已有记录）。`task` 工具仍在 schema 里；模型 `task schedule` 才会新建一条未来/周期任务。续跑由 SW alarms 或 UI `updateTask(resume)` 再 `sendMessage({ taskRun, taskId, taskContinuation: true })`，不写假用户消息。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Player-YN/BrowserKitten](https://github.com/Player-YN/BrowserKitten) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
