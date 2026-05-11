---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Codebase Search

This repository is indexed for semantic code search via the `claude-context` MCP server. Prefer `mcp__claude-context__search_code` (with path `D:\mcpmux`) over Grep/Glob when looking for implementations, understanding how features work, or finding related code across the codebase. Use Grep/Glob for exact string matches or filename patterns. If search returns an indexing error, re-index with `mcp__claude-context__index_codebase` first.

## What is McpMux

McpMux is a desktop app + local gateway that lets users configure MCP servers once and connect every AI client (Cursor, Claude Desktop, VS Code, Windsurf) through a single `localhost:45818` endpoint. Credentials are encrypted in the OS keychain instead of plain-text JSON files.

## Repository Structure

This is a multi-project workspace with 6 independent projects (not a unified monorepo):

| Project | Tech | Purpose |
|---------|------|---------|
| `mcp-mux/` | Tauri 2 (Rust + React 19), pnpm workspace | Desktop app + local MCP gateway |
| `mcpmux.bundler/` | Cloudflare Worker | GitHub webhook receiver that processes server definitions into D1/R2 |
| `mcpmux.serverhub.api/` | Cloudflare Worker (Hono) | REST API for server registry discovery (KV -> D1 -> R2 fallback chain) |
| `mcpmux.discover.ui/` | Next.js 16, shadcn/ui, Tailwind 4 | Web UI for browsing the server registry |
| `mcp-servers/` | JSON + AJV validation | Community MCP server definitions repository |
| `mcpmux.space/` | Docs | Documentation and design space |

## mcp-mux (Desktop App) - Main Project

### Build & Dev Commands

All commands run from `mcp-mux/`:

```bash
pnpm setup              # First-time dev environment setup (PowerShell)
pnpm dev                # Tauri desktop app dev mode (Rust + React hot-reload)
pnpm dev:web            # Web UI only (Vite, no Rust)
pnpm build              # Production Tauri build (all platforms)
```

### Testing

```bash
pnpm test               # All tests (Rust + TypeScript)
pnpm test:rust          # cargo nextest run --workspace
pnpm test:rust:unit     # cargo nextest run --workspace --lib
pnpm test:rust:int      # cargo nextest run -p tests
pnpm test:rust:doc      # cargo test --workspace --doc
pnpm test:ts            # vitest run -c tests/ts/vitest.config.ts
pnpm test:ts:watch      # vitest watch mode
pnpm test:e2e           # WebDriver IO desktop E2E (needs MCPMUX_REGISTRY_URL)
pnpm test:e2e:file      # Single E2E spec: pnpm test:e2e:file -- tests/e2e/specs/foo.ts
pnpm test:e2e:grep      # E2E by name: pnpm test:e2e:grep -- "test name"
pnpm test:e2e:web       # Playwright web UI E2E
pnpm test:coverage      # cargo llvm-cov + vitest coverage
```

### Linting & Validation

```bash
pnpm validate           # Full check: cargo fmt + clippy + check + eslint + typecheck
pnpm lint               # ESLint (recursive) + cargo clippy --workspace -- -D warnings
pnpm lint:fix           # Auto-fix lint issues
pnpm format             # prettier --write . && cargo fmt --all
pnpm format:check       # Check formatting without modifying
pnpm typecheck          # TypeScript type checking (recursive)
```

### Rust Crate Architecture

The Cargo workspace has 4 library crates + 1 app crate + 1 test crate:

- **mcpmux-core** (`crates/mcpmux-core/`) - Domain layer: entities (Space, InstalledServer, FeatureSet, Client), repository traits, domain services, application services with event emission, and the central EventBus
- **mcpmux-gateway** (`crates/mcpmux-gateway/`) - Axum HTTP gateway: routes MCP calls to correct servers, manages OAuth 2.1+PKCE token refresh, filters tools/resources/prompts based on FeatureSets, per-client access key auth, server connection pooling
- **mcpmux-storage** (`crates/mcpmux-storage/`) - SQLite persistence with AES-256-GCM field-level encryption via ring, typed credential rows (per-token encryption), DPAPI key storage on Windows (`keychain_dpapi.rs`), OS keychain on macOS/Linux via keyring crate, zeroize for secure memory clearing
- **mcpmux-mcp** (`crates/mcpmux-mcp/`) - MCP protocol client management using rmcp SDK
- **apps/desktop/src-tauri** - Tauri 2 app shell, Tauri commands, system tray, deep-link handler (`mcpmux://`)
- **tests/rust** - Integration test crate

Key patterns: event-driven architecture (EventBus), repository pattern (trait-based storage abstraction), service layer pattern with DI via ApplicationServices builders.

### Frontend Architecture

- **Entry**: `apps/desktop/src/main.tsx` -> `App.tsx`
- **State**: Zustand store (`stores/appStore.ts`)
- **Hooks**: `useServerManager` (server CRUD), `useSpaces` (workspace switching), `useDomainEvents` (Rust event listeners), `useDataSync` (data synchronization)
- **UI**: React 19 + Tailwind CSS + Lucide icons + Monaco Editor (config editing)
- **Path aliases**: `@/` -> `src/`, `@mcpmux/ui` -> shared UI package

### Data Flow

```
AI Clients -> McpMux Gateway (localhost:45818/mcp) -> MCP Servers (stdio/HTTP)
                     |
              Authenticates (access keys)
              Routes (per-space server config)
              Filters (FeatureSet permissions)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mcpmux/mcp-mux](https://github.com/mcpmux/mcp-mux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
