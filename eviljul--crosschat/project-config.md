---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

<!-- CODEGRAPH_START -->
## CodeGraph

This repository is indexed by CodeGraph (`.codegraph/` exists at the repo root). **Reach for it BEFORE grep/find or reading files when you need to understand or locate code:**

- **MCP tool** (when available): `codegraph_explore` answers most code questions in one call — the relevant symbols' verbatim source plus the call paths between them, including dynamic-dispatch hops grep can't follow. Name a file or symbol in the query to read its current line-numbered source. If it's listed but deferred, load it by name via tool search.
- **Shell** (always works): `codegraph explore "<symbol names or question>"` prints the same output.

If there is no `.codegraph/` directory, skip CodeGraph entirely — indexing is the user's decision.
<!-- CODEGRAPH_END -->

## 项目一句话

CrossChat 是基于 **Tauri 2 (Rust) + React 19 + TypeScript** 的跨 LLM 桌面端 AI 助手。**已完成向单一「六边形架构」的重写**：旧的双栈共存（Tauri command 全家桶 / 内嵌 HTTP 服务）**已从编译链移除**，当前只有一条激活路径。**该路径已不再是薄壳**：核心链路已接通 **OpenAI / Anthropic / DeepSeek** 三家适配器的**流式输出（SSE）+ 只读工具 ReAct 循环**，API Key 已迁至系统钥匙串（keyring）。六边形其余部分（MCP / 事件总线 / 写类工具审批）仍是「已搭好但未接线」的脚手架。

> ⚠️ **版本号与文档超前警告**：`Cargo.toml` / `package.json` / `tauri.conf.json` 均为 `0.2.3`，但 `CHANGELOG.md`（标 v0.3.0）、`QUICKSTART.md`、`docs/REFACTOR_*.md` 描述的 **Axum HTTP Server / SSE / 21434 端口 / sqlx / 10-50x 流式**，在当前代码里**并不存在**（`Cargo.toml` 无 `axum`、无 `sqlx`，`lib.rs` 无 `http_server` 模块）。**以代码为准，别信这些文档**。

## 开发命令

环境要求：Node.js 22+、Rust（`dtolnay/rust-toolchain@stable`，未锁版本）、Python 3.11（仅构建期打包 office 依赖用）。

```bash
# 启动开发（前端固定端口 5174 + Tauri 主进程）
npm install
npm run tauri dev

# 生产构建（产物在 src-tauri/target/release/bundle/）
npm run tauri build

# 仅前端
npm run dev
npm run build           # tsc 类型检查 + vite build
npm run preview

# Rust 检查
cd src-tauri
cargo check
cargo build

# 重新生成 Tauri 图标
npm run icon design/icon-1024.png   # 或 npx @tauri-apps/cli icon design/icon-1024.png
```

**没有前端测试框架、没有 lint 脚本**。改动 TS 至少跑 `npm run build`（含 `tsc`）；改动 Rust 跑 `cargo check`。
注意：仓库里 `*_tests.rs`（`agent/`、`mcp/`、`memory/`、`metrics/`、`skills/`）**大多位于已停用的死代码模块**，不参与编译，`cargo test` 跑不到它们。

## 架构现状（最关键认知）

**只有一条激活路径**，现已接通流式与只读工具调用：

```
CanvasView.tsx ─invoke("send_chat_message", Channel<StreamEvent>)─▶ commands/chat_cmd.rs
   │
   ├─ make_model_client ── 按 providerType 路由：
   │     openai-compat → OpenAIClient（SSE 真流式）
   │     anthropic     → AnthropicClient
   │     deepseek      → DeepSeekClient
   │     └─ ReAct 循环（MAX_TOOL_ITERATIONS）：流式生成 → 若请求工具则执行 → 回填 → 再生成
   ├─ LocalToolHost（adapters/tool）── 只读工具白名单：read_file / list_directory
   └─ SqliteThreadStore（rusqlite）── ~/.../.crosschat/threads.db（items 累积落单个 Turn）
```

- **发消息**：`chat_cmd::send_chat_message` 按 `providerType` 选择 ModelClient → **流式**生成（`tauri::ipc::Channel<StreamEvent>` 逐块回推前端）→ 进入 **ReAct 循环**（上限 `MAX_TOOL_ITERATIONS`）：模型若请求工具，就经 `LocalToolHost` 执行**只读工具**并把结果回填 messages，再次流式生成，直到某轮无工具调用为止 → 全过程 items（AssistantText / ToolCall / ToolResult…）累积落库到单个 Turn。
- **当前激活链已包含**：多 Provider 路由（OpenAI / Anthropic / DeepSeek）、真流式（SSE 逐块）、只读工具调用（`read_file` / `list_directory`）、ReAct 多轮循环、`<think>` 标签实时折叠、AI 回复 markdown 渲染。
- **当前激活链仍没有**：MCP、写类工具与审批门、上下文压缩、skill、向量记忆。
- **`lib.rs` 装配 8 个模块**：`core / ports / adapters / application / migration / commands / error / python_env`。
- **`invoke_handler` 注册 22 个 command**：原 15 个（含 `migrate_data`）+ 新增 `set_session_status` `rename_session` `set_session_pinned`（会话归档 / 重命名 / 置顶）、`set_api_key` `get_api_key` `delete_api_key`（系统钥匙串）、`ocr_image`（本地 OCR）。

## 后端地图（`src-tauri/src/`）

### 激活代码（真正编译、真正跑）

```
core/models/       # 纯数据：thread.rs / turn.rs / tool.rs / message.rs（无 I/O）
                   # Turn 用 #[serde(tag="type")] 的 TurnItem 枚举：UserMessage/AssistantText/
                   # AssistantReasoning/ToolCall/ToolResult/Compaction/Approval/Error
ports/             # 5 个 trait：ModelClient / ToolHost / ThreadStore / EventBus / ApprovalGate
                   # ModelClient / ToolHost / ThreadStore 已有激活实现方；EventBus / ApprovalGate 仍无
adapters/store/    # sqlite_store.rs = SqliteThreadStore（rusqlite）
                   # 三张表：threads / turns(data 存 Turn 的 JSON) / todos
adapters/model/    # openai_client / anthropic_client / deepseek_client —— 均已由 chat_cmd 接线激活
adapters/tool/     # local_tool_host.rs = LocalToolHost（只读工具 read_file / list_directory）——已激活
                   # mcp_persistent / sandbox（未被 chat_cmd 调用）
commands/          # mod.rs 声明 chat_cmd / file_ops / keychain_cmd / ocr_cmd / session_cmd
                   #   chat_cmd.rs     → send_chat_message（中枢：流式 + ReAct + 只读工具）、fetch_models
                   #   session_cmd.rs  → create/list/get/save/delete + 归档/重命名/置顶（读写 SqliteThreadStore）
                   #   file_ops.rs     → 目录/文件读写删（已加敏感路径黑名单沙箱，见「安全」）
                   #   keychain_cmd.rs → set/get/delete_api_key（keyring，API Key 存系统钥匙串）
                   #   ocr_cmd.rs      → ocr_image（调嵌入式 Python 跑 resources/ocr.py 做本地 OCR）
python_env.rs      # 嵌入式 Python 运行器（get_python_executable / is_python_available）——被 ocr_cmd 激活
migration.rs       # 旧 ~/.crosschat/sessions/*.json → 新 threads.db
                   # 首次启动后台 spawn，成功写 .migrated 标记，原文件备份到 sessions_backup/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EvilJul/CrossChat](https://github.com/EvilJul/CrossChat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
