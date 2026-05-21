---
trigger: always_on
description: > A lightweight desktop app for managing MCP (Model Context Protocol) servers.
---

# MCPDock — AI Agent Guide

> A lightweight desktop app for managing MCP (Model Context Protocol) servers.
> Built with **Tauri 2 + Vue 3 + Rust**.

---

## Project Overview

MCPDock lets users visually manage MCP servers, group them, and expose aggregated endpoints via a local Streamable HTTP gateway. Key features:

- **MCP Server Management** — Add STDIO or Streamable HTTP servers, connect/disconnect, discover tools/prompts/resources.
- **Group & Gateway** — Group servers into endpoints at `http://localhost:{port}/mcp/{group_name}` with optional Bearer auth.
- **Settings** — Gateway port, auth, timeout, keep-alive, proxy, language, theme, auto-start.
- **System Tray** — Close-to-tray, single instance, dock icon toggle (macOS).
- **i18n** — Full Chinese (zh-CN) and English support.
- **Auto-update** — Built-in updater via `tauri-plugin-updater`.

---

## Tech Stack

| Layer | Technology |
| --- | --- |
| Desktop Framework | Tauri 2 |
| Backend | Rust, axum, tokio, rmcp, rusqlite (bundled SQLite) |
| Frontend | Vue 3, TypeScript, `<script setup lang="ts">` |
| UI Components | Naive UI |
| Styling | Tailwind CSS 4 (no config file — tokens in `src/style.css` `@theme {}`) |
| State Management | Pinia |
| i18n | vue-i18n |
| Linting / Formatting | Biome |
| Package Manager | pnpm |
| Build | Vite + Rolldown |

---

## Architecture

```
mcpdock/
├── src/                        # Vue 3 frontend
│   ├── components/             # Page & layout components
│   │   ├── AppSidebar.vue      # Navigation sidebar (mcp / group / settings)
│   │   ├── PageHeader.vue      # Reusable page header
│   │   ├── GatewayStatus.vue   # Gateway status indicator
│   │   ├── McpManagement.vue   # MCP server list & management
│   │   ├── GroupManagement.vue # Group management
│   │   ├── SettingsPage.vue    # Application settings
│   │   ├── mcp/                # MCP sub-components
│   │   │   ├── McpServerList.vue
│   │   │   ├── McpServerCard.vue
│   │   │   ├── McpServerForm.vue
│   │   │   ├── McpImportView.vue
│   │   │   ├── McpToolRunner.vue
│   │   │   └── shared.ts
│   │   └── group/              # Group sub-components
│   │       ├── GroupCard.vue
│   │       ├── GroupForm.vue
│   │       └── GroupList.vue
│   ├── stores/                 # Pinia stores
│   │   ├── mcp.ts             # Server state, IPC, runtime events
│   │   ├── group.ts           # Group state, IPC
│   │   ├── settings.ts        # Settings state, theme, locale
│   │   └── updater.ts        # Auto-update state
│   ├── types/                  # Shared TypeScript interfaces
│   │   ├── mcp.ts
│   │   ├── group.ts
│   │   └── settings.ts
│   ├── i18n/                   # vue-i18n setup
│   ├── locales/                # Language packs (en.ts, zh-CN.ts)
│   ├── assets/                 # Static assets + design spec
│   ├── App.vue                 # Root layout (sidebar + page切换)
│   ├── main.ts                 # App entry (Naive UI plugin, Pinia, i18n)
│   └── style.css               # Global styles + Tailwind @theme tokens
├── src-tauri/                  # Rust backend
│   ├── src/
│   │   ├── main.rs             # Entry point
│   │   ├── lib.rs              # Tauri builder: setup, tray, single-instance, gateway
│   │   ├── state.rs            # AppState (DB, runtimes, clients, settings, gateway)
│   │   ├── process_env.rs      # PATH repair for child processes
│   │   ├── commands/           # Tauri IPC command handlers
│   │   │   ├── mcp.rs          # Server CRUD, connect/disconnect, tool call
│   │   │   ├── group.rs        # Group CRUD + gateway restart
│   │   │   ├── settings.rs     # Settings read/write + gateway restart
│   │   │   ├── gateway.rs      # Gateway status query & restart
│   │   │   ├── capability.rs   # Capability listing
│   │   │   ├── import_export.rs # JSON import/export
│   │   │   └── install_mode.rs # Windows portable detection
│   │   ├── mcp/                # MCP client management
│   │   │   ├── runtime.rs      # McpClientHolder, McpServerRuntime types
│   │   │   └── manager/        # Connection lifecycle
│   │   │       ├── mod.rs      # connect, disconnect, refresh, call_tool
│   │   │       ├── transport.rs # STDIO & Streamable HTTP client creation
│   │   │       ├── discovery.rs # Tool/prompt/resource discovery
│   │   │       └── runtime_state.rs # Runtime state & event emission
│   │   ├── gateway/            # Streamable HTTP gateway
│   │   │   ├── server.rs       # Axum server, auth middleware, CORS
│   │   │   └── handler/        # GroupHandler dispatch
│   │   │       ├── mod.rs
│   │   │       ├── tools.rs
│   │   │       ├── prompts.rs
│   │   │       └── resources.rs
│   │   └── db/                 # SQLite data layer
│   │       ├── mod.rs          # Schema init (WAL mode, foreign keys)
│   │       ├── mcp_server.rs   # Server CRUD
│   │       ├── mcp_group.rs    # Group CRUD
│   │       ├── mcp_capability.rs # Capability storage
│   │       ├── app_settings.rs # Key-value settings
│   │       └── error.rs        # DB error type
│   ├── capabilities/          # Tauri permission capabilities
│   ├── icons/                 # App icons (macOS, Windows, tray)
│   └── Cargo.toml
├── biome.json                  # Biome config (lint, format, CSS)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hsingjui/mcpdock](https://github.com/hsingjui/mcpdock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
