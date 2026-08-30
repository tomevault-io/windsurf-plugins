---
trigger: always_on
description: Memoh is a multi-member, structured long-memory AI agent platform with isolated workspace runtimes. Users can create AI bots and chat with them via Telegram, Discord, Lark (Feishu), DingTalk, WeChat, Matrix, Email, and more. Each bot can use an independent container workspace to edit files, execute commands, run tools, and build itself while keeping runtime ownership explicit.
---

# AGENTS.md

## Project Overview

Memoh is a multi-member, structured long-memory AI agent platform with isolated workspace runtimes. Users can create AI bots and chat with them via Telegram, Discord, Lark (Feishu), DingTalk, WeChat, Matrix, Email, and more. Each bot can use an independent container workspace to edit files, execute commands, run tools, and build itself while keeping runtime ownership explicit.

The public documentation site is maintained separately in `felinics/memoh-docs`.

## Architecture Overview

Deploy/server mode consists of three core services:

| Service | Tech Stack | Port | Description |
|---------|-----------|------|-------------|
| **Server** (Backend) | Go + Echo | 8080 | Main service: REST API, auth, database, container management, **in-process AI agent** |
| **Channel** (Backend) | Go + Echo | 8081 | External channel adapters, email delivery, and channel webhook endpoints; delegates agent turns to Server over authenticated internal gRPC. Split mode is opt-in via `internal_rpc.shared_secret` (docker compose sets it); without it the Server embeds the channel runtime and runs all-in-one |
| **Web** (Frontend) | Vue 3 + Vite | 8082 | Management UI: visual configuration for Bots, Models, Channels, etc. |

The native desktop client is a separate distribution boundary for Memoh Cloud or a hosted Memoh server. `apps/desktop` reuses `@memohai/web` modules, but owns the Electron shell, system tray behavior, menus, preload IPC, cache invalidation, and packaged application resources.

Infrastructure dependencies:
- **PostgreSQL** — Relational data storage
- **Qdrant** — Vector database for memory semantic search
- **Workspace runtime** — Isolated containers per bot via Docker, containerd v2, or Apple Virtualization

## Tech Stack

### Backend (Go)
- **Framework**: Echo (HTTP)
- **Dependency Injection**: Uber FX
- **AI SDK**: [Twilight AI](https://github.com/felinics/twilight) (Go LLM SDK — OpenAI, Anthropic, Google)
- **Database Driver**: pgx/v5 (PostgreSQL)
- **Code Generation**: sqlc (SQL → Go)
- **API Docs**: Swagger/OpenAPI (swaggo)
- **MCP**: modelcontextprotocol/go-sdk
- **Containers / Workspaces**: Docker / containerd v2 / Apple Virtualization adapters

### Frontend (TypeScript)
- **Framework**: Vue 3 (Composition API)
- **Build Tool**: Vite 8
- **State Management**: Pinia 3 + Pinia Colada
- **UI**: Tailwind CSS 4 + custom component library (`@felinic/ui`) + Reka UI
- **Icons**: lucide-vue-next + `@memohai/icon` (brand/provider icons)
- **i18n**: vue-i18n
- **Markdown**: markstream-vue + Shiki + Mermaid + KaTeX
- **Desktop**: Electron 34 + [electron-vite](https://electron-vite.github.io/) 4 native client, reusing `@memohai/web` modules while managing the desktop renderer, tray behavior, menus, and preload IPC
- **Package Manager**: pnpm monorepo

### Tooling
- **Task Runner**: mise
- **Package Managers**: pnpm (frontend monorepo), Go modules (backend)
- **Linting**: golangci-lint (Go), ESLint + typescript-eslint + vue-eslint-parser (TypeScript)
- **Testing**: Vitest
- **Version Management**: bumpp
- **SDK Generation**: @hey-api/openapi-ts (with `@hey-api/client-fetch` + `@pinia/colada` plugins)

## Project Structure

```
Memoh/
├── cmd/                        # Go application entry points
│   ├── agent/                  #   Main backend server (embedded all-in-one or split-mode server process)
│   ├── channel/                #   Standalone channel service (split mode): platform adapters, email, webhooks
│   ├── internal/               #   Shared FX composition roots
│   │   ├── core/               #     Core (agent/db/api) module assembly
│   │   └── channel/            #     Channel runtime module assembly (ServerLocal / Runtime / Embedded)
│   ├── bridge/                 #   In-container gRPC bridge (UDS-based, runs inside bot containers; supervises optional display/browser helpers)
│   │   └── template/           #     Prompt templates for bridge (TOOLS.md, SOUL.md, IDENTITY.md, etc.)
│   ├── gen-bridge-mtls/        #   Bridge mTLS certificate generator
│   ├── mcp/                    #   MCP stdio transport binary
│   └── synccaps/               #   Build-time sync of provider template capabilities from the LiteLLM registry
├── internal/                   # Go backend core code (domain packages)
│   ├── accounts/               #   User account management (CRUD, password hashing)
│   ├── acl/                    #   Access control list (source-aware chat trigger ACL)
│   ├── arch/                   #   Architecture guard tests (channel-boundary import rules, spec §8)
│   ├── agent/                  #   Agent bounded package; root is a namespace, not a Go package
│   │   ├── adapter/            #     Adapters from lower-level domain ports to runtime implementations
│   │   ├── application/        #     Turn orchestration: context, persistence, memory, approvals, runtime dispatch

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [felinics/Memoh](https://github.com/felinics/Memoh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
