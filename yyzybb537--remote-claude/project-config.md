---
trigger: always_on
description: This file provides guidance to Claude-Code/Codex when working with code in this repository.
---

# CLAUDE.md/AGENTS.md

This file provides guidance to Claude-Code/Codex when working with code in this repository.

# **关键语言要求**
你必须完全使用 **简体中文** 进行交互、思考和汇报。

## 项目概述

Remote Claude 是一个双端共享 Claude/Codex CLI 工具。通过 PTY + Unix Socket 架构，支持多个终端客户端和飞书客户端并发连接同一个 Claude 或 Codex 会话，实现协作式 AI 对话。

## 架构

```
Claude/Codex CLI (PTY)
      │
  server.py         ← PTY 代理，管理进程/控制权/历史缓存
      │
  Unix Socket (/tmp/remote-claude/<name>.sock)
      │
  ┌───┴────┐
  │        │
client.py  SessionBridge (lark_client/)
(终端)     (飞书机器人)
```

**核心模块：**
- `remote_claude.py` — CLI 入口，子命令：start / attach / list / kill / lark
- `server/server.py` — PTY 代理服务器，`pty.fork()` 启动 Claude/Codex，asyncio Unix Socket 广播输出
- `server/parsers/claude_parser.py` — Claude CLI 终端输出解析（区域切分、Block 分类、执行状态判断）
- `server/parsers/codex_parser.py` — Codex CLI 终端输出解析（无分割线、`›` 提示符、背景色区域检测）
- `server/component_parser.py` — 向后兼容 shim（实际实现在 `server/parsers/`）
- `server/shared_state.py` — 共享内存写入（`.mq` 文件）
- `client/client.py` — 终端客户端，raw mode 输入转发
- `utils/protocol.py` — 消息协议（JSON + `\n` 分隔，二进制数据 base64 编码）。7 种消息类型：INPUT / OUTPUT / CONTROL / STATUS / HISTORY / ERROR / RESIZE
- `utils/session.py` — socket 路径管理、会话生命周期
- `utils/components.py` — 控制权状态机，SHARED（默认，所有人可输入）和 EXCLUSIVE（独占）两种模式

**飞书客户端 (`lark_client/`)：**
- `main.py` — WebSocket 入口，事件分发
- `lark_handler.py` — 命令路由，以 `chat_id` 为 key 统一管理群聊/私聊的 bridge 和绑定
- `session_bridge.py` — 连接 Unix Socket，**仅负责输入发送**（send_input/send_key）和连接管理
- `shared_memory_poller.py` — **流式滚动卡片轮询器**：每秒轮询 `.mq` 共享内存，通过 hash diff 驱动 `CardSlice`/`StreamTracker` 就地更新或冻结+开新卡
- `card_builder.py` — **`build_stream_card(blocks, status_line, bottom_bar, is_frozen, agent_panel, option_block, session_name, disconnected)`**：四层结构卡片构建（内容区/状态区/交互区/菜单）+ 辅助卡片（session_list/menu/help/dir 等）
- `card_service.py` — 飞书卡片 API 服务（create/update/send）
- `rich_text_renderer.py` — 持久化 pyte Screen 封装（server 端实时喂入）

**Server 端数据流（全量快照架构）：**
```
PTY data → self._renderer.feed(data) → HistoryScreen(220×100, history=5000) 持久化实时更新
                                            ↓ SU/SD 正确执行（ESC[nS/ESC[nT）
                                            ↓ 滚出行 → history.top（最多 5000 行）
                                            ↓（flush 触发）
                            （开启 --debug-screen）_write_screen_debug(原始 screen) → _screen.log
                                            ↓
                                    VirtualScreen(history.top + screen.buffer)
                                            ↓
                                    ScreenParser.parse(vscreen)
                                            ↓
                                    raw components 列表
                                            ↓
                                    分拣：visible_blocks / status_line / bottom_bar
                                            ↓
                                 ┌── 时序窗口平滑 ──┐
                                 │ _FrameObs 记录原始值 │
                                 │ 平滑 status_line     │
                                 │ 平滑 block blink     │
                                 └────────┬──────────┘
                                          ↓
                                    all_blocks = visible_blocks
                                          ↓
                                    ClaudeWindow 快照
                                          ↓
                                ├→ _messages.log (debug)
                                └→ .mq 共享内存 (全量覆写)
```

> `_screen.log` 只在 CLI 传入 `--debug-screen` 时写入：每次 flush 触发后、正式解析 `pyte.Screen` 之前，`server/server.py` 会覆盖输出当前屏幕快照到 `/tmp/remote-claude/<name>_screen.log`，方便比对 blink 标记、行号与布局切分。

**输出处理管道（流式滚动卡片模型）：**
```
Server (SharedStateWriter) → .mq 文件（ClaudeWindow 全量快照）
                                 ↑ 每秒轮询
SharedMemoryPoller          → StreamTracker（流式跟踪状态）
                                 ↓ hash diff
CardService                 → 同一张卡片就地更新 / 超限时冻结+开新卡
```

核心理念：没有 turn、没有 message。只有一个不断增长的 blocks 流和跟踪它的滚动窗口。

**职责分界（强制约束）：**

| 层 | 职责 | 禁止事项 |
|----|------|---------|
| **server.py**（服务端） | 保证写入共享内存（`.mq` 文件）的 Claude 会话输出**完整、准确**；负责 ANSI 解析、终端状态还原、消息结构化 | — |
| **lark_client/**（飞书客户端） | 从共享内存内容到飞书卡片渲染的**纯展示流程** | **严禁**对内容做字符串修复、ANSI 清理、格式补全等处理；若内容有误，应修 server 端而非在客户端打补丁 |

> **原则：** 飞书客户端拿到的数据应该是已经可以直接渲染的干净内容。任何"内容不对"的问题，根因在 server 端，修复也在 server 端。

**关键设计决策：**
- **tmux 环境变量修复**：server 运行在 detached tmux 会话中，tmux 会覆盖 `TERM_PROGRAM`（改为 `tmux`）并设置 `TMUX`/`TMUX_PANE`，导致 Claude CLI 的 Ink 框架判定不支持 kitty keyboard protocol，Shift+Enter 退化为 Enter。修复方案：`cmd_start` 在构建 server_cmd 时通过 env prefix 注入原始终端变量（`TERM_PROGRAM`/`TERM_PROGRAM_VERSION`/`COLORTERM`）；`_start_pty` 在 PTY 子进程中清除 `TMUX`/`TMUX_PANE`
- **输入端**（`_forward_to_claude` / `handle_option_select`）只调用 `bridge.send_input/send_key`，不创建卡片
- **输出端**完全由 `SharedMemoryPoller` 驱动：attach 时启动轮询，detach/断线时停止
- **流式滚动窗口**：`StreamTracker` 跟踪 blocks 流，`CardSlice` 记录每张卡片的窗口位置（`start_idx`）
- **首次 attach**：取最近 `INITIAL_WINDOW=30` 个 blocks 渲染到一张卡片，更早内容通过 `/history` 查看
- **卡片超限**：`len(blocks) - start_idx > MAX_CARD_BLOCKS=50` 时冻结当前卡（灰色 header、移除状态区和按钮区），从冻结位置之后开新卡
- **群聊/私聊统一**：`_bridges[chat_id]` 和 `_chat_sessions[chat_id]` 统一管理，无需分组
- **降级机制**：update_card 失败时创建新卡片，更新 CardSlice 中的 card_id，sequence 归零
- **持久化绑定**：`~/.remote-claude/lark_chat_bindings.json`（chat_id → session_name）


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yyzybb537/remote_claude](https://github.com/yyzybb537/remote_claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
