---
trigger: always_on
description: > 面向 **Cursor**、**Codex** 及其他 AI coding agent 的本仓库工作指南。
---

# Vibe Pet — Agent Guide

> 面向 **Cursor**、**Codex** 及其他 AI coding agent 的本仓库工作指南。  
> 修改代码前先读本文；协议细节见 `docs/`。

## 快速索引

| 项目 | 值 |
| --- | --- |
| 产品名 | Vibe Pet |
| npm 包名 | `vibe-pet` |
| 当前版本 | `1.7.0` |
| 运行时 | Electron + Node.js ≥ 18 |
| 主入口 | `src/desktop/main.js` |
| 本地 bridge 默认端口 | `17384` |
| 运行时端口文件 | `~/.code-pet/runtime.json` |
| 协议文档 | [IDE/Agent](docs/ide-protocol.md) · [硬件](docs/hardware-protocol.md) |

## 架构

```text
IDE/CLI hooks ──┐
Codex JSONL ────┼──► StateHub (src/host/state-hub.js)
Cursor DB/logs ─┘         │
                          ├──► 主窗口桌宠网格 (src/desktop/)
                          ├──► 桌面浮动桌宠 (pet-overlay)
                          ├──► BLE 写入 (Electron 主进程)
                          └──► GET /api/device-snapshot (Wi-Fi 轮询)
```

- **桌面桥**：Electron 主进程启动 `src/host/` HTTP 服务，负责 hook 上报、状态聚合、Petdex 代理；渲染进程负责 UI 与 BLE。
- **StateHub**：会话 key 为 `agentId:sessionId`；多会话聚合后下发硬件。
- **Petdex**：角色 manifest 来自 `https://petdex.dev/api/manifest`，经 `/api/petdex/manifest` 缓存代理。

参考项目：[clawd-on-desk](https://github.com/rullerzhou-afk/clawd-on-desk) 的状态映射思路。

## 目录结构

```text
src/desktop/                         Electron 桌面应用（主窗口 + 浮动桌宠 + BLE + 烧录）
src/host/                            本地 HTTP bridge 与 StateHub
src/host/agent/                      后台监听器（Codex JSONL / Cursor composer / transcript / presence）
src/host/public/                     备用 Web Bluetooth 桥 UI
src/hooks/                           各 agent 的 hook / plugin 脚本
  agent-hook.js                      通用 CLI hook 入口
  cursor-hook.js                     Cursor 专用 hook
  codex-hook.js                      Codex 专用 hook
src/scripts/                         安装、打包、检查、固件打包脚本
src/firmware/
  wio-terminal-code-pet/             Wio Terminal（BLE）
  esp-ai-mini-ext-tft-code-pet/      ESP-AI Mini Ext TFT（BLE）
  sensecap-indicator-code-pet/       SenseCAP Indicator（BLE）
docs/                                协议与贡献文档
scripts/install.{sh,ps1,cmd}         跨平台一键安装
```

## 常用命令

```bash
npm install              # 安装依赖并同步 hooks/plugins
npm start                # 启动桌面应用
npm run dev              # 热更新开发（监听 src/desktop + src/host）
npm run check            # 项目检查
npm run install:hooks    # 手动同步 hooks/plugins
npm run uninstall:hooks  # 卸载 hooks/plugins
npm run bridge:web       # 仅启动 Web 桥（无 Electron）
npm run firmware:package # 打包各硬件 main.bin
npm run build            # electron-builder 安装包（当前平台）
npm run package:current  # Electron Packager 未签名应用包 → dist/
```

端口覆盖：`CODE_PET_PORT=17385 npm start` 或 `npm start -- --port 17385`

跳过 hook 同步：`VIBE_PET_SKIP_HOOKS=1 npm start`

## 给 Cursor 的工作说明

### 集成位置

| 项 | 路径 |
| --- | --- |
| Hook 配置 | `~/.cursor/hooks.json` |
| Hook 脚本 | `src/hooks/cursor-hook.js` |
| 安装逻辑 | `src/scripts/install-hooks.js` → `registerCursor()` |

### Hook 事件（已注册）

`sessionStart` · `sessionEnd` · `beforeSubmitPrompt` · `preToolUse` · `postToolUse` · `postToolUseFailure` · `subagentStart` · `subagentStop` · `preCompact` · `afterAgentThought` · `stop`

### 额外状态来源（无需 hook 也能部分工作）

| 监听器 | 文件 | source 标识 |
| --- | --- | --- |
| Composer DB | `src/host/agent/cursor-composer-monitor.js` | `cursor-composer` |
| Transcript | `src/host/agent/cursor-transcript-monitor.js` | `cursor-transcript` |

### Cursor hook → 桌宠状态

| Hook 事件 | 状态 |
| --- | --- |
| `sessionStart` | `idle` |
| `sessionEnd` | `sleeping` |
| `beforeSubmitPrompt` | `thinking` |
| `preToolUse` | `working` |
| `postToolUse` | `thinking` |
| `postToolUseFailure` | `error` |
| `subagentStart` | `juggling` |
| `preCompact` | `sweeping` |
| `stop` | `attention` |

### 改 Cursor 集成时

- 改事件映射：优先改 `src/hooks/cursor-hook.js` 的 `HOOK_TO_STATE`
- 增删 hook 事件：同步改 `install-hooks.js` 的 `CURSOR_EVENTS`
- 不要破坏 `beforeSubmitPrompt` 需返回 `{"continue":true}` 的行为

## 给 Codex 的工作说明

### 集成位置

| 项 | 路径 |
| --- | --- |
| Hook 配置 | `~/.codex/hooks.json` |
| Feature 开关 | `~/.codex/config.toml` → `[features] hooks = true` |
| Hook 脚本 | `src/hooks/codex-hook.js` |
| JSONL 监听 | `src/host/agent/codex-log-monitor.js` |

### Hook 事件（已注册）

`SessionStart` · `UserPromptSubmit` · `PreToolUse` · `PermissionRequest` · `PostToolUse` · `Stop`

### JSONL 回退

- 路径：`~/.codex/sessions/**/rollout-*.jsonl`
- 当 official hook 未覆盖或延迟时，由 `CodexLogMonitor` 补状态
- 已被 official hook 覆盖的事件见 `state-hub.js` 中 `CODEX_LOG_EVENTS_COVERED_BY_OFFICIAL_HOOKS`

### Codex hook → 桌宠状态

| Hook 事件 | 状态 |
| --- | --- |
| `SessionStart` | `idle` |
| `UserPromptSubmit` | `thinking` |
| `PreToolUse` | `working` |
| `PermissionRequest` | `notification` |
| `PostToolUse` | `thinking` |
| `Stop` | `attention` |

### JSONL 常见事件 → 状态

| JSONL 事件 | 状态 |
| --- | --- |
| `event_msg:task_started` | `thinking` |
| `response_item:function_call` | `working` |
| `event_msg:task_complete` | `attention` → `idle` |
| `event_msg:context_compacted` | `sweeping` |
| 权限相关 | `notification` |

### 改 Codex 集成时

- Hook 映射：`src/hooks/codex-hook.js` 的 `EVENT_TO_STATE`
- JSONL 映射：`src/host/agent/codex-log-monitor.js`
- 安装后用户可能需在 Codex CLI 执行 `/hooks` 并批准新命令

## 其他 Agent 集成

`npm install` / `npm start` / `npm run dev` 会自动调用 `install-hooks.js`。未安装对应 agent 时跳过。

| Agent | 方式 | 配置位置 | agentId |
| --- | --- | --- | --- |
| Windsurf | hooks | `~/.codeium/windsurf/hooks.json` | `windsurf` |
| Claude CLI / Claude Code | hooks | `~/.claude/settings.json` | `claude-code` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Seeed-Solution/vibe-pet](https://github.com/Seeed-Solution/vibe-pet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
