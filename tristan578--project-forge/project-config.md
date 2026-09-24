---
trigger: always_on
description: SpawnForge is a browser-based, AI-native 2D/3D game engine. It is a polyglot monorepo:
---

# SpawnForge — Copilot Instructions

## Project Overview

SpawnForge is a browser-based, AI-native 2D/3D game engine. It is a polyglot monorepo:

- **engine/** — Rust (Bevy 0.18) compiled to WebAssembly via wasm-bindgen. Pure game logic in `engine/src/core/`, JS interop bridge in `engine/src/bridge/`.
- **web/** — TypeScript/React (Next.js 16) editor frontend. State via Zustand store slices. Strict TypeScript, zero ESLint warnings.
- **mcp-server/** — TypeScript MCP server exposing engine commands as AI-callable tools via WebSocket.

## Build & Test Commands

```bash
# Web — install deps, lint, typecheck, unit tests (run from web/)
cd web && npm install  # use npm ci in CI environments
npx eslint --max-warnings 0 .
npx tsc --noEmit
npx vitest run

# MCP server tests (run from mcp-server/)
cd mcp-server && npm install
npx vitest run

# WASM engine build (requires Rust stable + wasm32-unknown-unknown target + wasm-bindgen-cli 0.2.127)
cd engine && cargo build --target wasm32-unknown-unknown --release --features webgl2
cd engine && cargo build --target wasm32-unknown-unknown --release --features webgpu
```

## Architecture Rules

- **Bridge isolation**: Only `engine/src/bridge/` may import `web_sys`/`js_sys`/`wasm_bindgen`. `core/` is pure Rust.
- **Command-driven**: All engine operations go through `handle_command()` JSON commands. Never create UI-only shortcuts that bypass the command system.
- **Event-driven**: Bevy → bridge → JS callback → Zustand store → React re-render.
- **Store slices**: Each Zustand concern gets its own slice. Use selectors to prevent unnecessary re-renders.
- **WASM calls**: Only `web/src/hooks/useEngine.ts` may make direct WASM calls.

## Coding Standards

### Rust (engine/)
- Target: `wasm32-unknown-unknown`. Never use `std::fs`, `std::net`, or other non-WASM APIs.
- All `unsafe` blocks MUST have a `// SAFETY:` comment.
- Use `serde` with `serde_wasm_bindgen` for JS ↔ Rust serialization.
- Prefer `Result<T, E>` over panics. Feature flags: `webgl2` and `webgpu`.

### TypeScript (web/, mcp-server/)
- Strict mode. Never use `any`. Avoid `as` casts.
- All chat handler arguments MUST be validated before use (manual validation via `parseArgs()` pattern — Zod is not used for runtime validation).
- Use named exports. Prefer `const` over `let`. Never use `var`.
- Tailwind CSS for all styling. No inline styles or CSS modules.

## MANDATORY: Taskboard-Driven Development

**No code changes without a ticket.** This is enforced by hooks on all AI tools.

Before writing ANY code:
1. Check the taskboard at http://localhost:3010 (API: http://localhost:3010/api)
2. Pick an existing ticket OR create a new one with ALL required fields
3. Move the ticket to `in_progress`

Required ticket fields: User Story, Description (20+ chars), Acceptance Criteria (3+ Given/When/Then scenarios), Priority, Team (Engineering or PM — use the team IDs in **Canonical Project Facts** below), Subtasks (3+).

<!-- AGENTIC-SYNC:START -->
<!-- Generated from tools/agentic-sync/canonical.json by tools/agentic-sync/sync.mjs.
     Do NOT hand-edit between these markers — edit canonical.json and run
     `node tools/agentic-sync/sync.mjs --write`. CI gate: scripts/check-agentic-sync.sh. -->

### Canonical Project Facts

**Taskboard** — the single source of truth for all work:
- Project: **Project Forge** (`01KMM9ZA6SBZ7RKJZJTZS9VR4R`, prefix `PF`)
- Teams: Engineering `01KMR5E36TP59PRQA8GQEWJVM1`, PM `01KMR5E3852BWXAZ219W47CSKS`
- API: `http://localhost:3010/api` · Web UI: `http://localhost:3010`
- Start: `taskboard start --port 3010`  *(do not pass `--db` — it uses the OS-default DB)*
- These IDs are board-local; if a query 404s, rediscover with `curl -s http://localhost:3010/api/projects`

**Pinned versions:** Next.js 16.3.5 · React 19.3.0 · wasm-bindgen 0.2.127 · Bevy 0.18 *(wasm-bindgen must match Cargo.lock exactly)*

**Coverage thresholds (CI-enforced):** statements 85 · branches 77 · functions 80 · lines 86

**Quick validation:** `cd web && npx eslint --max-warnings 0 . && npx tsc --noEmit && npx vitest run`
<!-- AGENTIC-SYNC:END -->

See `AGENTS.md` for full taskboard setup, workflow, and GitHub Project sync details.

### Sync Architecture (Non-Negotiable)
- `github_issue_number` (SQLite column) is the SOLE link between local tickets and GitHub Issues. NEVER match by title.
- `sync_repo` (SQLite column) controls which repo a ticket syncs to. Only `sync_repo = 'project-forge'` tickets are synced.
- The JSON map file is a cache. SQLite columns are authoritative.
- Tickets from other local projects are NEVER synced (they have `sync_repo = NULL`).

## Security

- All chat input passes through `sanitizeChatInput()` in `web/src/lib/chat/sanitizer.ts`. Never bypass.
- API keys encrypted with AES-256-GCM in `web/src/lib/encryption.ts`. Never log or expose.
- All API routes require auth via `web/src/lib/auth/api-auth.ts`.
- Never commit secrets. Use `.env.local` (gitignored).
- **Worktree safety:** When working in a git worktree, commit after every logical chunk of work. Agent processes can be terminated at any time — uncommitted work is permanently lost. The stop hook (`.claude/hooks/on-stop.sh`) auto-commits as a safety net, but agents MUST commit frequently themselves.

## CI/CD Enforcement


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tristan578/project-forge](https://github.com/Tristan578/project-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
