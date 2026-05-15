---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Sharpclaw is an advanced AI assistant framework built on **.NET 10**, featuring a robust **cross-conversation long-term memory system** and **system-level operation capabilities**. It serves as an autonomous AI agent that can explore, plan, and execute complex tasks in real codebases.

The project consists of two main components:
1. **Sharpclaw** — The AI agent framework with multi-channel frontend support
2. **Sharc** — A high-performance, pure managed C# library for reading/writing SQLite files (submodule)

---

## Build & Run Commands

```bash
# Build everything (including sharc submodule)
dotnet build

# Run Sharpclaw (Web 宿主, 默认模式)
dotnet run --project sharpclaw                              # 启动 Web 宿主 (含 WebSocket + 可选 QQBot)
dotnet run --project sharpclaw web                          # 同上
dotnet run --project sharpclaw web --address 0.0.0.0 --port 8080

# CLI 客户端 (连接到 Web 宿主)
dotnet run --project sharpclaw cli                          # 连接到默认 localhost:5000
dotnet run --project sharpclaw cli --address 192.168.1.2 --port 8080

# TUI 模式 (本地独立运行)
dotnet run --project sharpclaw tui                          # TUI mode (Terminal.Gui)
dotnet run --project sharpclaw config                       # Re-run config dialog
dotnet run --project sharpclaw help                         # Show usage info

# Web 模式需要 config 已存在; CLI 需先启动 Web 宿主
# TUI 首次运行时自动弹出配置对话框
```

### Sharc Submodule Commands

```bash
# Build sharc only
dotnet build sharpclaw/sharc/Sharc.sln

# Run sharc tests
dotnet test sharpclaw/sharc/tests/Sharc.Tests
dotnet test sharpclaw/sharc/tests/Sharc.IntegrationTests
dotnet test sharpclaw/sharc                              # All tests

# Run sharc benchmarks (NEVER run full suite, use small chunks)
dotnet run -c Release --project sharpclaw/sharc/bench/Sharc.Comparisons -- --filter '*CoreBenchmarks*SequentialScan*'
dotnet run -c Release --project sharpclaw/sharc/bench/Sharc.Comparisons -- --tier mini
```

---

## Architecture

### Sharpclaw Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  Web 宿主 (ASP.NET Core, 默认启动模式)                        │
│  ├── /    → Web UI (index.html)                             │
│  ├── /ws  → WebSocket 端点 (多客户端)                        │
│  │         ├── Web UI 浏览器客户端                            │
│  │         └── CLI 终端客户端 (CliClient)                     │
│  └── QQBotHostedService (IHostedService, 可选)               │
├─────────────────────────────────────────────────────────────┤
│  独立模式                                                     │
│  ├── Tui/ — Terminal.Gui v2 (本地运行, ChatWindow)           │
│  └── Cli/ — CliClient (WebSocket 客户端, 连接到 Web 宿主)     │
├─────────────────────────────────────────────────────────────┤
│  Agent Layer (Agents/)                                       │
│  ├── MainAgent — Conversation loop, tool orchestration      │
│  ├── MemorySaver — Autonomous memory management             │
│  └── ConversationArchiver — Two-phase memory consolidation  │
├─────────────────────────────────────────────────────────────┤
│  Memory Pipeline (Chat/, Memory/)                            │
│  ├── MemoryPipelineChatReducer — Context window management  │
│  ├── VectorMemoryStore — Sharc + SQLite vector search       │
│  └── InMemoryMemoryStore — Keyword-based fallback           │
├─────────────────────────────────────────────────────────────┤
│  Command System (Commands/)                                  │
│  ├── FileCommands — File operations (cat, edit, find, etc.) │
│  ├── ProcessCommands — Bash/PowerShell execution            │
│  ├── HttpCommands — HTTP requests                           │
│  ├── TaskCommands — Background task management              │
│  └── SystemCommands — System info, exit                     │
├─────────────────────────────────────────────────────────────┤
│  Core Infrastructure (Core/)                                 │
│  ├── AgentBootstrap — Shared initialization                 │
│  ├── SharpclawConfig — Configuration with encryption        │
│  ├── ClientFactory — LLM client creation                    │
│  ├── DataProtector/KeyStore — AES-256-GCM encryption        │
│  └── TaskManager — Background process management            │
└─────────────────────────────────────────────────────────────┘
```

### Multi-Tier Memory System

Sharpclaw implements a sophisticated three-layer memory pipeline:

| Layer | File | Purpose | Written By |
|-------|------|---------|------------|
| **Working Memory** | `working_memory.json` | Current conversation snapshot (JSON) | MainAgent (each turn) |
| **Recent Memory** | `recent_memory.md` | Detailed summaries (append-only) | ConversationArchiver (Summarizer) |
| **Primary Memory** | `primary_memory.md` | Consolidated core facts | ConversationArchiver (Consolidator) |
| **Vector Store** | `memories.db` | Semantic embeddings + metadata | VectorMemoryStore |
| **History** | `history/*.md` | Archived full conversations | ConversationArchiver |

**Memory Pipeline Flow:**
1. After each turn → MemorySaver analyzes and updates vector store

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imxcstar/sharpclaw](https://github.com/imxcstar/sharpclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
