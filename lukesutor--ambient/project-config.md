---
trigger: always_on
description: Ambient is a local-first AI desktop assistant built with **Tauri 2.x** (Rust + Next.js 15). It features an agentic runtime with tool-calling capabilities, local inference via llama.cpp, optional cloud fallback to Gemini, BYOK (Bring Your Own Key) support for OpenAI/Gemini/Anthropic-compatible APIs, and browser-use capabilities. The architecture prioritizes privacy, extensibility, and reliability.
---

# Ambient AI Assistant - Project Guidelines

## Project Overview

Ambient is a local-first AI desktop assistant built with **Tauri 2.x** (Rust + Next.js 15). It features an agentic runtime with tool-calling capabilities, local inference via llama.cpp, optional cloud fallback to Gemini, BYOK (Bring Your Own Key) support for OpenAI/Gemini/Anthropic-compatible APIs, and browser-use capabilities. The architecture prioritizes privacy, extensibility, and reliability.

**Tech Stack:**
- Backend: Rust with Tauri 2.x
- Frontend: Next.js 15 (SSG mode), React 19, TypeScript
- UI: shadcn/ui (Radix primitives) + Tailwind CSS v4
- Local LLM: llama.cpp server (ships with Qwen3VL-2B)
- Cloud LLM: Gemini via Cloudflare Worker + BYOK (OpenAI, Gemini, Anthropic compatible)
- Database: SQLCipher (encrypted SQLite) with rusqlite + sqlite-vec for embeddings
- State: React Context + useReducer pattern

## Code Style

### TypeScript/React
- Components: `PascalCase.tsx` in [src/components/](app/src/components/)
- Use `"use client"` directive only when hooks/interactivity needed (see [layout.tsx](app/src/app/layout.tsx))
- Import order: React/Next → third-party → @/components → @/types → @/lib
- Component structure: types → component → hooks → effects → handlers → render
- File naming: `kebab-case.tsx` for files, `PascalCase` for component names
- shadcn/ui pattern: Use `cn()` utility for conditional classes (see [app-sidebar.tsx](app/src/components/app-sidebar.tsx))

### Rust
- Naming: `snake_case` modules/functions, `PascalCase` structs/enums, `SCREAMING_SNAKE_CASE` constants
- Error handling: Custom error enums with `thiserror`, convert to `String` for Tauri commands
- Logging: Prefix with module name: `log::info!("[module_name] Context message")`
- Module structure: `mod.rs` (re-exports), `types.rs`, `commands.rs`, service files (see [agents/](app/src-tauri/src/agents/))

## Architecture

### Frontend State Management

**Provider Hierarchy** ([AppProvider.tsx](app/src/lib/providers/AppProvider.tsx)):
```
SettingsProvider → RoleAccessProvider → ModelAccessProvider → SetupProvider → WindowsProvider → ConversationProvider
```

**Pattern:** Context + useReducer for all state management
- Define state interface + discriminated union actions
- Create reducer with switch statement
- Provider sets up event listeners for Tauri events via `listen()`
- Export custom hook that validates context exists

**Key Providers:**
- [ConversationProvider](app/src/lib/conversations/ConversationProvider.tsx): Chat messages, streaming, attachments
- [SettingsProvider](app/src/lib/settings/SettingsProvider.tsx): User settings, HUD dimensions
- [WindowsProvider](app/src/lib/windows/WindowsProvider.tsx): Window expand/collapse state
- [RoleAccessProvider](app/src/lib/role-access/RoleAccessProvider.tsx): Auth state, user info, Google auth status
- [SetupProvider](app/src/lib/setup/SetupProvider.tsx): Model download progress
- [ModelAccessProvider](app/src/lib/model-access/ModelAccessProvider.tsx): Credit usage, model list, user tier — updates in real time via Tauri events (`cloud_usage_decremented`, `models_changed`, `auth_changed`)

### Backend Module Organization

**Top-level modules** ([lib.rs](app/src-tauri/src/lib.rs)):
- `auth/`: OAuth flow, split token storage (refresh tokens in OS keyring, session tokens AES-encrypted in per-user store.json)
- `db/`: Per-user encrypted SQLite (SQLCipher) with migrations, conversations, messages, memory, token usage
- `agents/`: Chat runtime + browser-use runtime
- `models/`: LLM client (local/cloud providers), llama.cpp server, embedding, OCR
- `skills/`: Registry, executor, builtin skills (web-search, code-execution, memory)
- `events/`: Global emitter, typed event payloads
- `settings/`: User preferences, agent runtime config
- `windows/`: Window management, screen capture

**Module Pattern:**
```
module/
├── mod.rs          # Re-exports
├── types.rs        # Structs/enums
├── commands.rs     # Tauri commands
└── service.rs      # Business logic
```

### Agentic Chat Runtime

**Location:** [agents/chat/runtime.rs](app/src-tauri/src/agents/chat/runtime.rs)

**Loop Architecture:**
1. For cloud models: create a generation session via `/v1/usage/start-turn` (checks rate limit, increments usage once)
2. Get conversation history (context-limited based on local vs cloud)
3. Build `LlmRequest` with system prompt, messages, available tools, session token
4. Generate response via provider (local or cloud)
5. If text response → save message and return
6. If tool calls → execute in parallel, save results as messages, loop continues
7. Repeat until text response or max iterations reached
8. Check cancellation signal (`Arc<AtomicBool>`) on each iteration

**Session-Based Credit System (Cloud Models):**
- All cloud models draw from a shared daily credit pool (not per-model limits)
- Credit costs: Flash = 1 credit, Pro = 3 credits (decimal costs supported via `f64`/`REAL`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LukeSutor/ambient](https://github.com/LukeSutor/ambient) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
