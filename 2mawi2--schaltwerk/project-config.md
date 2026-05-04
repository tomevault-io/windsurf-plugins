---
trigger: always_on
description: Tauri-based desktop app for managing AI coding sessions using git worktrees. Each session gets an isolated branch/worktree where AI agents (Claude, GitHub Copilot CLI, Kilo Code, Gemini, OpenCode, Codex, Factory Droid, etc.) can work without affecting the main codebase.
---

# CLAUDE.md - Schaltwerk Development Guidelines

## Project Overview
Tauri-based desktop app for managing AI coding sessions using git worktrees. Each session gets an isolated branch/worktree where AI agents (Claude, GitHub Copilot CLI, Kilo Code, Gemini, OpenCode, Codex, Factory Droid, etc.) can work without affecting the main codebase.

## Platform Support
- macOS 11+ supported; Linux beta builds ship via releases.
- Windows 10 version 1903+ supported (ConPTY required); WSL not yet supported.

> **Tooling Note:** Examples in this guide default to `bun`. Replace them with the equivalent `npm` commands (`npm install`, `npm run …`, etc.) if you prefer npm.

## Working Directory (CRITICAL)

**Your starting working directory is where you work. Do not navigate away from it.**

- Check `<env>` for your working directory and current branch
- If in a worktree (branch: `schaltwerk/*`): All files are here. Make changes here.
- If in main repo (branch: `main`): Work here directly.
- Do NOT infer parent paths from directory structure
- Do NOT `cd` to other locations unless explicitly required

❌ WRONG: `cd /inferred/path && command`
✅ RIGHT: `command` (in current directory)

## System Architecture

### Core Concepts
- **Sessions**: Isolated git worktrees for AI agents to work in
- **Specs**: Draft/planning sessions without worktrees (can be converted to running sessions)
- **Orchestrator**: Special session that works directly in main repo (for planning/coordination)
- **Terminals**: Each session gets 2 PTY terminals (top/bottom) for running agents
- **Domains**: Business logic is organized in `src-tauri/src/domains/` - all new features should create appropriate domain modules, if there are legacy business domains duplicated they should be merged via scout rule into the new structure.

### Key Data Flows

**Session Creation → Agent Startup:**
1. `App.tsx:handleCreateSession()` → Tauri command `schaltwerk_core_create_session`
2. `domains/sessions/service.rs:SessionManager::create_session()` → Creates DB entry + worktree
3. Frontend switches via `SelectionContext` → Lazy terminal creation
4. Agent starts in terminal with worktree as working directory

**MCP API → Session Management:**
- External tools call REST API (port 8547+hash) → Creates/updates specs
- Backend emits `SessionsRefreshed` event → UI updates automatically
- Optional `Selection` event → UI switches to new session

**Session State Transitions:**
- Spec → Running: `start_spec_session()` creates worktree + terminals
- Running → Reviewed: `mark_session_reviewed()` flags for merge
- Running → Spec: `convert_to_spec()` removes worktree, keeps content

### Critical Files to Know

**Frontend Entry Points:**
- `App.tsx`: Main orchestration, session management, agent startup
- `SelectionContext.tsx`: Controls which session/terminals are active

**Backend Core:**
- `main.rs`: Tauri commands entry point
- `schaltwerk_core/mod.rs`: Session gateway + database access
- `domains/terminal/manager.rs`: PTY lifecycle management
- `domains/git/worktrees.rs`: Git worktree operations

**Communication Layer:**
- `eventSystem.ts`: Type-safe frontend event handling
- `events.rs`: Backend event emission
- `mcp_api.rs`: REST API for external MCP clients

### State Management (MANDATORY)
- Shared UI/application state lives in Jotai atoms under `src/store/atoms`; expose read-only atoms plus action atoms when updates require side effects.
- Example: `src/store/atoms/fontSize.ts` stores terminal/UI font sizes, updates CSS variables, emits `UiEvent.FontSizeChanged`, and persists via `SchaltwerkCoreSetFontSizes`.
- Reach for Jotai when state crosses components, needs persistence, or must be accessed from tests using the Jotai `Provider`/`createStore`; keep purely local state in React `useState`.
- Access atoms with `useAtomValue`, `useSetAtom`, or `useAtom` instead of creating new context providers for the same data.

## Essential Commands

### Before Completing ANY Task
```bash
just test          # Run ALL validations: TypeScript, Rust lints, tests, and build
# Or: bun run test  # Same as 'just test'
```
**Why:** Ensures code quality and prevents broken commits. The script runs TypeScript lint + type-checking, MCP lint/tests, frontend vitest, Rust clippy, dependency hygiene (`cargo shear`), `knip`, Rust tests (`cargo nextest`), and a Rust build.

### Autonomy for Tests (MANDATORY)
- Codex and Factory Droid may run `just test`, `bun run test`, `bun run lint`, `bun run lint:rust`, `bun run test:rust`, and `cargo` checks without asking for user approval, even when the CLI approval mode is set to “on-request”.
- Rationale: Running the full validation suite is required to keep the repository green and accelerate iteration. Do not pause to request permission before executing these commands.

### Development Commands
```bash
# Starting Development
bun run tauri:dev       # Start app in development mode with hot reload
RUST_LOG=schaltwerk=debug bun run tauri:dev  # With debug logging

# Testing & Validation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [2mawi2/schaltwerk](https://github.com/2mawi2/schaltwerk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
