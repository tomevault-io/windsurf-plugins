---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

Clawd 桌宠 — 一个 Electron 桌面宠物，通过 hook 系统和日志轮询实时感知 AI coding agent 的工作状态并播放对应的像素风 SVG 动画。支持 **Claude Code**（command + HTTP hook）、**Codex CLI**（JSONL 日志轮询）、**Copilot CLI**（command hook）、**Cursor Agent**（`~/.cursor/hooks.json`，stdin JSON + stdout JSON）、**Gemini CLI**（session JSON 轮询）、**opencode**（in-process plugin + 反向 HTTP bridge）并行运行。支持 Windows、macOS 和 Linux。

## 常用命令

```bash
npm start              # 启动 Electron 应用（开发模式）
npm run build          # electron-builder 打包 Windows NSIS 安装包
npm run build:mac      # electron-builder 打包 macOS DMG（x64 + arm64）
npm run build:linux    # electron-builder 打包 Linux AppImage + deb
npm run build:all      # 同时打包 Windows + macOS + Linux
npm install            # 安装依赖（electron + electron-builder）
node hooks/install.js       # 注册 Claude Code hooks 到 ~/.claude/settings.json
npm run install:cursor-hooks # 注册 Cursor Agent hooks 到 ~/.cursor/hooks.json
npm run install:gemini-hooks # 注册 Gemini CLI hooks 到 ~/.gemini/settings.json
npm test               # 运行单元测试（node --test test/*.test.js）
```

手动测试状态切换：
```bash
curl -X POST http://127.0.0.1:23333/state \
  -H "Content-Type: application/json" \
  -d '{"state":"working","svg":"clawd-working-building.svg"}'
```

Shell 测试脚本（仅开发用，不随仓库分发）：
```bash
bash test-demo.sh [秒] # 逐个播放所有 SVG 动画（默认每个 8 秒）
bash test-mini.sh [秒] # 逐个播放极简模式 SVG 动画（默认每个 6 秒）
bash test-sleep.sh     # 缩短睡眠超时快速测试睡眠序列
bash test-bubble.sh    # 发送模拟权限请求测试气泡堆叠
bash test-macos.sh     # macOS 适配测试（需先 npm start）
```

单元测试覆盖 agents/、hook 注册和端口发现逻辑（`test/registry.test.js`、`test/codex-log-monitor.test.js`、`test/gemini-log-monitor.test.js`、`test/gemini-install.test.js`、`test/install.test.js`、`test/server-config.test.js`、`test/menu-autostart.test.js`），使用 Node.js 内置 test runner。Electron 主进程（状态机、窗口、托盘）无自动化测试，依赖手动 + shell 脚本验证。

## 架构与数据流

```
Claude Code 状态同步（command hook，非阻塞）：
  Claude Code 触发事件
    → hooks/clawd-hook.js（零依赖 Node 脚本，stdin 读 JSON 取 session_id + source_pid）
    → HTTP POST 127.0.0.1:23333/state { state, session_id, event, source_pid, cwd }
    → src/server.js 路由 → src/state.js 状态机（多会话追踪 + 优先级 + 最小显示时长 + 睡眠序列）
    → IPC state-change 事件
    → src/renderer.js（<object> SVG 预加载 + 淡入切换 + 眼球追踪）

Copilot CLI 状态同步（command hook，非阻塞）：
  Copilot 触发事件
    → hooks/copilot-hook.js（camelCase 事件名 → agents/copilot-cli.js 映射 → HTTP POST）
    → 同上状态机

Cursor Agent 状态同步（command hook，stdin JSON，非阻塞）：
  Cursor IDE 触发事件
    → hooks/cursor-hook.js（hook_event_name → 映射为 PascalCase event + HTTP POST，stdout 返回 allow/continue 以满足 preToolUse 等 hook）
    → 同上状态机（agent_id: cursor-agent）

Codex CLI 状态同步（JSONL 日志轮询，~1.5s 延迟）：
  Codex 写入 ~/.codex/sessions/YYYY/MM/DD/rollout-*.jsonl
    → agents/codex-log-monitor.js（增量读取，事件类型 → agents/codex.js 映射）
    → 同上状态机

Gemini CLI 状态同步（session JSON 轮询，~1.5s 延迟 + 4s 完成延迟窗口）：
  Gemini 写入 ~/.gemini/tmp/<project>/chats/session-*.json
    → agents/gemini-log-monitor.js（轮询 JSON，diff 消息数组检测工具调用/完成）
    → 同上状态机（agent_id: gemini-cli）

opencode 状态同步（in-process plugin，~0ms 延迟）：
  opencode 触发事件（session.created / session.status / message.part.updated 等）
    → hooks/opencode-plugin/index.mjs（Bun 运行时，插件跑在 opencode.exe 进程内）
    → translateEvent 映射（opencode v2 事件名 → PascalCase Clawd event 名）
    → fire-and-forget HTTP POST 127.0.0.1:23333/state
    → 同上状态机（agent_id: opencode）

opencode 权限气泡（event hook + 反向 bridge，非阻塞）：
  opencode 请求权限 → event hook 收到 permission.asked
    → plugin POST /permission（带 bridge_url + bridge_token）→ Clawd 立即 200 ACK（不挂连接）
    → Clawd 创建 bubble 窗口 → 用户 Allow/Always/Deny
    → Clawd POST plugin 的反向 bridge → bridge 用 ctx.client._client.post() 调 opencode 内置 Hono 路由 /permission/:id/reply
    → opencode 执行对应行为（once/always/reject）

远程 SSH 状态同步（反向端口转发）：
  远程服务器上的 Claude Code / Codex CLI
    → hooks 通过 SSH 隧道 POST 到本地 127.0.0.1:23333
    → 同上状态机（CLAWD_REMOTE=1 模式跳过 PID 收集）

权限决策流（Claude Code HTTP hook，阻塞）：
  Claude Code PermissionRequest
    → HTTP POST 127.0.0.1:23333/permission { tool_name, tool_input, session_id, permission_suggestions }
    → main.js 创建 bubble 窗口（bubble.html）显示权限卡片
    → 用户点击 Allow / Deny / suggestion → HTTP 响应 { behavior }
    → Claude Code 执行对应行为
```

### 双窗口架构（输入/渲染分离）

桌宠使用两个独立的顶层窗口：
- **渲染窗口（win）**：透明大窗口，永久 `setIgnoreMouseEvents(true)`（click-through），只负责显示 SVG 动画和眼球追踪
- **输入窗口（hitWin）**：小矩形窗口，`transparent: true` + `setShape` 覆盖 hitbox 区域，`focusable: true`，永久 `setIgnoreMouseEvents(false)`，接收所有 pointer 事件

输入事件流：hitWin renderer → IPC → main（移动两个窗口 + relay）→ renderWin renderer（播放反应动画）

这个架构解决了 Windows 上的拖拽失效 bug：`WS_EX_NOACTIVATE`（`setFocusable(false)`）+ layered window + Chromium child HWND 的组合，在 z-order 变化后会导致 click 走 WM_MOUSEACTIVATE 激活死路径。分离后输入窗口 `focusable: true` 避免了这个问题。

### 多 Agent 架构（agents/）

每个 agent 定义为一个配置模块，导出事件映射、进程名、能力声明：
- `agents/claude-code.js` — Claude Code 事件映射 + 能力（hooks、permission、terminal focus）
- `agents/codex.js` — Codex CLI JSONL 事件映射 + 轮询配置
- `agents/copilot-cli.js` — Copilot CLI camelCase 事件映射
- `agents/cursor-agent.js` — Cursor Agent（hooks.json）事件映射
- `agents/gemini-cli.js` — Gemini CLI 事件映射 + JSON 轮询配置
- `agents/opencode.js` — opencode 事件映射 + 能力（plugin、permission、terminal focus）
- `agents/registry.js` — agent 注册表：按 ID 或进程名查找 agent 配置

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yx0716/clawd-insights](https://github.com/yx0716/clawd-insights) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
