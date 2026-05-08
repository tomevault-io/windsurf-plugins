---
trigger: always_on
description: Cortex Desktop is an AI-powered development environment (IDE) built with Tauri v2. It provides a modern GUI for AI coding agents with features including an integrated terminal, LSP support, debugger (DAP), Git integration, MCP context servers, extension hosting, and multi-provider AI chat/agent orchestration. The frontend is built with SolidJS and the backend is Rust.
---

# AGENTS.md — Cortex Desktop

## Project Purpose

Cortex Desktop is an AI-powered development environment (IDE) built with Tauri v2. It provides a modern GUI for AI coding agents with features including an integrated terminal, LSP support, debugger (DAP), Git integration, MCP context servers, extension hosting, and multi-provider AI chat/agent orchestration. The frontend is built with SolidJS and the backend is Rust.

## Architecture Overview

```
┌──────────────────────────────────────────────────────────────────┐
│  Frontend (SolidJS + TypeScript)                                 │
│  src/                                                            │
│  ├── components/   797 UI component files (editor, terminal, git, etc.)│
│  ├── context/      181 context files (99 top-level + sub-contexts)    │
│  ├── hooks/        Custom SolidJS hooks                          │
│  ├── pages/        Route pages (Home, Session, Admin, Share)     │
│  ├── providers/    Monaco editor providers (LSP bridge)          │
│  ├── sdk/          Tauri IPC client SDK (client.ts, executor.ts) │
│  ├── services/     Business logic services                       │
│  └── design-system/ Design tokens and primitives                 │
├──────────────────────────────────────────────────────────────────┤
│  Tauri IPC Bridge (invoke commands + emit/listen events)         │
├──────────────────────────────────────────────────────────────────┤
│  Backend (Rust / Tauri v2)                                       │
│  src-tauri/src/                                                  │
│  ├── ai/           AI provider management, agent orchestration   │
│  ├── lsp/          Language Server Protocol client                │
│  ├── dap/          Debug Adapter Protocol client                 │
│  ├── terminal/     PTY terminal management + shell integration   │
│  ├── git/          Git operations via libgit2 (29 submodules)    │
│  ├── fs/           File system operations + caching              │
│  ├── extensions/   VS Code-compatible extension system           │
│  ├── remote/       SSH remote development                        │
│  ├── factory/      Agent workflow orchestration (designer/exec)  │
│  ├── collab/       Real-time collaboration (CRDT, WebSocket)     │
│  ├── mcp/          Model Context Protocol server                 │
│  ├── timeline/     Local file history tracking                   │
│  ├── acp/          Agent Control Protocol tools                  │
│  ├── editor/       Editor features (folding, symbols, refactor)  │
│  ├── i18n/         Internationalization (locale detection)       │
│  ├── settings/     User/workspace settings persistence           │
│  └── ...           48 modules total (150-line lib.rs)            │
├──────────────────────────────────────────────────────────────────┤
│  Sidecar Services                                                │
│  └── mcp-server/   MCP stdio server (TypeScript/Node.js)         │
├──────────────────────────────────────────────────────────────────┤
│  Local Engine Modules (formerly from cortex-cli)                  │
│  ├── cortex_engine    (Config, Session, SessionHandle, SSRF)     │
│  ├── cortex_protocol  (Event/Submission types, policies)         │
│  └── cortex_storage   (Session persistence, message history)     │
└──────────────────────────────────────────────────────────────────┘
```

**Data Flow:** Frontend components dispatch Tauri IPC `invoke()` calls → Rust `#[tauri::command]` handlers process requests → Results returned as JSON. Real-time events use Tauri's `emit()`/`listen()` event system (e.g., terminal output, LSP diagnostics, AI streaming). The SDK layer (`src/sdk/`) wraps all IPC calls with typed functions and error handling.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend Framework | SolidJS 1.9.11 with TypeScript 5.9 |
| UI Components | Kobalte (headless), custom design system |
| Styling | Tailwind CSS v4.1.18 |
| Code Editor | Monaco Editor 0.55.1 |
| Terminal | xterm.js 6.0 with WebGL renderer |
| Bundler | Vite 7.3 with vite-plugin-solid |
| Testing (Frontend) | Vitest 3.2 with jsdom |
| Testing Library | @solidjs/testing-library 0.8 |
| Desktop Framework | Tauri v2.9 (Rust) |
| Rust Edition | 2024 (rust-version 1.85, nightly required) |
| Async Runtime | Tokio (full features) |
| Database | SQLite via rusqlite (bundled) |
| Git | libgit2 via git2 crate |
| Serialization | serde + serde_json + rmp-serde (MessagePack) |
| Security | keyring, secrecy, zeroize for credential management |
| Syntax Highlighting | Shiki 3.21+ |
| MCP Server | @modelcontextprotocol/sdk + zod |

## Critical Rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CortexLM/cortex-ide](https://github.com/CortexLM/cortex-ide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
