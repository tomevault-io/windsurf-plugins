---
trigger: always_on
description: AgentStage is a Windows desktop multi-agent roleplay chat app built with **Tauri v2** (Rust backend + WebView2 frontend). The frontend uses **Svelte 5 + Vite + TailwindCSS v4**; the backend uses **Rust + SQLite (rusqlite)**. All LLM API calls are proxied through the Rust backend—**never from the frontend**.
---

# AgentStage — Agent Quick Reference

AgentStage is a Windows desktop multi-agent roleplay chat app built with **Tauri v2** (Rust backend + WebView2 frontend). The frontend uses **Svelte 5 + Vite + TailwindCSS v4**; the backend uses **Rust + SQLite (rusqlite)**. All LLM API calls are proxied through the Rust backend—**never from the frontend**.

---

## Development Commands

```bash
# Full dev environment (starts Vite + compiles Rust + opens window)
pnpm tauri dev

# Frontend build only
pnpm build

# Rust type check only (~3s)
cd src-tauri
cargo check

# Run Rust backend only (no Vite frontend)
cd src-tauri
cargo run

# Svelte type check
npx svelte-check --tsconfig ./tsconfig.json
```

> **Important:** `pnpm dev` alone starts Vite but not the Rust backend. Always use `pnpm tauri dev` for full-stack development.

---

## Project Boundaries

| Directory | Role |
|-----------|------|
| `src/` | Frontend: Svelte 5 components, stores (`.svelte.ts`), types |
| `src-tauri/src/` | Rust backend: Tauri Commands, DB repositories, models, crypto |
| `src-tauri/src/db/` | SQLite connection, schema, migrations, handwritten repositories |
| `src-tauri/src/commands/` | Tauri IPC command handlers (exposed to frontend via `invoke`) |
| `src-tauri/src/models/` | Rust structs for DB rows and request/response DTOs |
| `docs/` | Product docs: PRD.md, feature_list.md, schema.md, tech-stack.md |

---

## Frontend Traps (Svelte 5 + Tailwind v4)

### Mount syntax
Svelte 5 uses `mount()`, not `new App()`:
```ts
import { mount } from 'svelte';
const app = mount(App, { target: document.getElementById('app')! });
```

### `tsconfig.json` — `useDefineForClassFields` must be `false`
Svelte 5 Runes (`$state`) inside classes will break at runtime if this is `true`:
```json
"useDefineForClassFields": false
```

### TailwindCSS v4 syntax
Use `@import "tailwindcss"` and `@theme` in `styles.css`. Do **not** use `@tailwind base/components/utilities` or `tailwind.config.js`.
Custom colors are defined in `@theme`:
```css
@theme {
  --color-primary: #3b82f6;
  --color-bg: #f3f4f6;
}
```

### Svelte `class:` directive does not support `/`
Class names with opacity modifiers (e.g. `bg-primary/10`) cannot be used with Svelte's `class:` directive. Use inline conditional strings instead:
```svelte
<!-- Wrong -->
<div class:bg-primary/10={active} />

<!-- Right -->
<div class={active ? 'bg-primary/10 text-primary' : ''} />
```

### State management
Use Svelte 5 Runes in `.svelte.ts` files. No Redux/Zustand needed. Example:
```ts
// src/lib/stores/appState.svelte.ts
class AppState {
    currentView = $state<'agents' | 'chat' | 'history'>('agents');
    selectedAgentId = $state<string | null>(null);
}
export const appState = new AppState();
```

---

## Backend Traps (Rust + SQLite)

### Database location
SQLite file is created at runtime in the program directory (forced portable mode):
```
<exe_dir>\data\agentstage.db
```
In dev mode, data is stored at the project root `data/`.
WAL mode is enforced (`PRAGMA journal_mode = WAL`).

### Async mutex for DB connection
The `DbState` wraps the `rusqlite::Connection` in a `tokio::sync::Mutex`. **Never** use `std::sync::Mutex` in async Tauri commands.

### No ORM — handwritten SQL
All queries are raw SQL in repository modules (`src/db/*.rs`). Schema changes require:
1. Update `src/db/schema.rs` (DDL)
2. Add a migration in `src/db/migration.rs`
3. Update the corresponding repository CRUD methods

### API Key security
- API Keys are encrypted with AES-256-GCM in Rust (`src/crypto.rs`)
- `AgentResponse` DTO **excludes** `api_key_encrypted` — it never leaves the backend
- Frontend sends the raw key only during create/update; backend encrypts before storage

### LLM calls go through Rust
Frontend **must not** call OpenAI/Claude APIs directly. All LLM interactions are Tauri Commands that the Rust backend executes. This protects API keys and prevents Prompt inspection via DevTools.

---

## Tauri IPC Design

Commands are registered in `src/lib.rs` via `tauri::generate_handler!`. Current commands (in `src/commands/agent.rs`):
- `create_agent`
- `get_agent`
- `list_agents`
- `update_agent`
- `delete_agent` (soft delete)

Frontend calls them with:
```ts
import { invoke } from '@tauri-apps/api/core';
const agents = await invoke<Agent[]>('list_agents');
```

### Parameter naming: camelCase in frontend, snake_case in Rust

Tauri v2's `#[tauri::command]` macro **automatically converts** camelCase (frontend) ↔ snake_case (Rust). The frontend sends camelCase; the macro deserializes it into snake_case Rust parameters.

```rust
// Rust (src-tauri)
#[tauri::command]
pub async fn update_quiet_hours(quiet_hours_start: i32, quiet_hours_end: i32) -> Result<(), String> { ... }
```

```ts
// Frontend (src) — MUST use camelCase
await invoke('update_quiet_hours', { quietHoursStart: 0, quietHoursEnd: 480 });
```

> **Rule:** Frontend `invoke()` calls always use **camelCase** parameter keys. Rust command parameters are always **snake_case**. Tauri v2 bridges them automatically.

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SupaNeko/AgentStage](https://github.com/SupaNeko/AgentStage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
