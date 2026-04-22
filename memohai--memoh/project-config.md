---
trigger: always_on
description: Memoh is a multi-member, structured long-memory, containerized AI agent system platform. Users can create AI bots and chat with them via Telegram, Discord, Lark (Feishu), DingTalk, WeChat, Matrix, Email, and more. Every bot has an independent container and memory system, allowing it to edit files, execute commands, and build itself — providing a secure, flexible, and scalable solution for multi-bot management.
---

# AGENTS.md

## Project Overview

Memoh is a multi-member, structured long-memory, containerized AI agent system platform. Users can create AI bots and chat with them via Telegram, Discord, Lark (Feishu), DingTalk, WeChat, Matrix, Email, and more. Every bot has an independent container and memory system, allowing it to edit files, execute commands, and build itself — providing a secure, flexible, and scalable solution for multi-bot management.

## Architecture Overview

The system consists of three core services:

| Service | Tech Stack | Port | Description |
|---------|-----------|------|-------------|
| **Server** (Backend) | Go + Echo | 8080 | Main service: REST API, auth, database, container management, **in-process AI agent** |
| **Web** (Frontend) | Vue 3 + Vite | 8082 | Management UI: visual configuration for Bots, Models, Channels, etc. |
| **Browser Gateway** | Bun + Elysia + Playwright | 8083 | Browser automation service: headless browser actions for bots |

Infrastructure dependencies:
- **PostgreSQL** — Relational data storage
- **Qdrant** — Vector database for memory semantic search
- **Containerd** — Container runtime providing isolated environments per bot (Linux); Apple Virtualization on macOS

## Tech Stack

### Backend (Go)
- **Framework**: Echo (HTTP)
- **Dependency Injection**: Uber FX
- **AI SDK**: [Twilight AI](https://github.com/memohai/twilight-ai) (Go LLM SDK — OpenAI, Anthropic, Google)
- **Database Driver**: pgx/v5
- **Code Generation**: sqlc (SQL → Go)
- **API Docs**: Swagger/OpenAPI (swaggo)
- **MCP**: modelcontextprotocol/go-sdk
- **Containers**: containerd v2 (Linux), Apple Virtualization (macOS)
- **TUI**: Charm libraries (bubbletea, glamour, lipgloss) for CLI interactive mode

### Frontend (TypeScript)
- **Framework**: Vue 3 (Composition API)
- **Build Tool**: Vite 8
- **State Management**: Pinia 3 + Pinia Colada
- **UI**: Tailwind CSS 4 + custom component library (`@memohai/ui`) + Reka UI
- **Icons**: lucide-vue-next + `@memohai/icon` (brand/provider icons)
- **i18n**: vue-i18n
- **Markdown**: markstream-vue + Shiki + Mermaid + KaTeX
- **Desktop**: Tauri (wraps `@memohai/web`)
- **Package Manager**: pnpm monorepo

### Browser Gateway (TypeScript)
- **Runtime**: Bun
- **Framework**: Elysia
- **Browser Automation**: Playwright

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
│   ├── agent/                  #   Main backend server (main.go, FX wiring)
│   ├── bridge/                 #   In-container gRPC bridge (UDS-based, runs inside bot containers)
│   │   └── template/           #     Prompt templates for bridge (TOOLS.md, SOUL.md, IDENTITY.md, etc.)
│   ├── mcp/                    #   MCP stdio transport binary
│   └── memoh/                  #   Unified CLI (Cobra: serve, migrate, chat, bots, compose, docker, login, install, support)
├── internal/                   # Go backend core code (domain packages)
│   ├── accounts/               #   User account management (CRUD, password hashing)
│   ├── acl/                    #   Access control list (source-aware chat trigger ACL)
│   ├── agent/                  #   In-process AI agent (Twilight AI SDK integration)
│   │   ├── agent.go            #     Core agent: Stream() / Generate() via Twilight SDK
│   │   ├── stream.go           #     Streaming event assembly
│   │   ├── sential.go          #     Sential (sentinel) loop detection logic
│   │   ├── prompt.go           #     Prompt assembly (system, heartbeat, schedule, subagent, discuss)
│   │   ├── config.go           #     Agent service dependencies
│   │   ├── types.go            #     Shared types (StreamEvent, GenerateResult, FileAttachment)
│   │   ├── fs.go               #     Filesystem utilities
│   │   ├── guard_state.go      #     Guard state management
│   │   ├── retry.go            #     Retry logic
│   │   ├── read_media.go       #     Media reading utilities
│   │   ├── spawn_adapter.go    #     Spawn adapter for sub-processes
│   │   ├── prompts/            #     Prompt templates (Markdown, with partials prefixed by _)
│   │   │   ├── system_chat.md, system_discuss.md, system_heartbeat.md, system_schedule.md, system_subagent.md
│   │   │   ├── _tools.md, _memory.md, _contacts.md, _schedule_task.md, _subagent.md
│   │   │   ├── heartbeat.md, schedule.md
│   │   │   └── memory_extract.md, memory_update.md
│   │   └── tools/              #     Tool providers (ToolProvider interface)
│   │       ├── message.go      #       Send message tool
│   │       ├── contacts.go     #       Contact list tool

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [memohai/Memoh](https://github.com/memohai/Memoh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
