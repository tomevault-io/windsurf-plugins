---
trigger: always_on
description: > **Purpose:** This document governs all development work on `zosma-cowork`. Every agent must read and follow these standards before writing code.
---

# Zosma Cowork — Agent Engineering Standards

> **Purpose:** This document governs all development work on `zosma-cowork`. Every agent must read and follow these standards before writing code.

---

## 1. Philosophy

- **TDD First:** No production code without a failing test first. The only exceptions are trivial config/string changes (Scenario 3).
- **Less Code, More Work:** Reusable components, pure functions, and shared utilities over duplication.
- **Modular Architecture:** Each module has a single responsibility. Dependencies flow inward.
- **Production Ready:** Every PR must pass CI (test, lint, build, security scan) before merge.

---

## 2. Test-Driven Development (TDD)

### The Iron Law

```
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
```

### Three Scenarios

| Scenario | When | Process |
|---|---|---|
| **1. New Feature** | Default for all new files/functions | 1. Write failing test → 2. Watch it fail → 3. Write minimal code → 4. Watch it pass → 5. Refactor |
| **2. Modify Tested Code** | Changing code with existing tests | 1. Run existing tests (green) → 2. Make change → 3. Run tests (still green) → 4. Add test if behavior changed |
| **3. Trivial Change** | Typo, config tweak, string literal | Use judgment. Run relevant tests after. |

### Test Requirements

- One behavior per test. If the name contains "and", split it.
- Use real code over mocks. Mock only external I/O (filesystem, network, Tauri APIs).
- Watch the test **fail for the expected reason** before implementing.
- All tests must pass with pristine output (no warnings, no errors).

### Testing Stack

| Layer | Framework | Command |
|---|---|---|
| Frontend | Vitest + React Testing Library + `@testing-library/jest-dom` | `npm test` |
| Agent Sidecar | TypeScript `tsc --noEmit` | `cd agent-sidecar && npx tsc --noEmit` |
| Tauri Relay | `cargo test` | `cargo test --workspace` |

---

## 3. Code Architecture

### Frontend (React + TypeScript)

```
src/
├── components/ui/          # shadcn-style primitives (Button, Badge, etc.)
│   └── NEVER import app-specific code
├── components/             # App-specific components
│   ├── ActivityBar.tsx
│   ├── ChatMessage.tsx
│   ├── MessageInput.tsx
│   ├── Sidebar.tsx
│   └── WelcomeScreen.tsx
├── hooks/                  # Reusable React hooks
│   └── usePiStatus.ts
├── lib/                    # Pure utilities (no React)
│   └── utils.ts            # cn(), formatters, parsers
├── types/                  # Shared TypeScript types
│   └── index.ts
├── services/               # Business logic / API wrappers (future)
│   └── pi-client.ts
└── store/                  # State management (future)
    └── session-store.ts
```

**Rules:**
- `components/ui/` are pure, reusable primitives. They know nothing about pi.
- `lib/` contains pure functions with zero side effects. These are the easiest to test.
- `hooks/` wrap imperative APIs (Tauri invoke, browser APIs) into declarative React.
- `services/` encapsulate all Tauri command calls and external I/O.

### Backend (Tauri Relay — Rust)

```
src-tauri/src/
├── main.rs                 # Entry point only
└── lib.rs                  # Tauri commands, sidecar process management
```

**Rules:**
- The Tauri relay is intentionally thin — it spawns the Node.js sidecar and forwards JSON commands.
- All agent logic lives in `agent-sidecar/`.
- Tauri commands return `Result<T, String>` with user-friendly error messages.

### Agent Sidecar (Node.js)

```
agent-sidecar/src/
└── index.ts                # pi-mono SDK: auth, models, sessions, extensions
```

**Rules:**
- Communicates with Tauri via stdin/stdout JSON lines.
- Uses `@earendil-works/pi-coding-agent` (pi-mono SDK) for all agent capabilities.
- Never write to stdout except JSON protocol messages (logging goes to stderr).

---

## 4. Styling & UI

- **shadcn/ui** primitives with Tailwind CSS v4.
- Theme tokens via CSS custom properties in `App.css`.
- Dark mode only (for now). All colors reference `hsl(var(--...))`.
- Desktop-native feel: minimal borders, subtle shadows, proper scrollbars.

---

## 5. Git Workflow

### Branch Naming

```
feat/session-persistence
fix/memory-leak-in-chat
refactor/extract-pi-client
docs/api-reference
chore/update-dependencies
```

### Commit Convention (Conventional Commits)

```
feat: add session persistence via Tauri events
fix: prevent double scroll on message append
refactor: extract PiClient from usePiStatus hook
test: add coverage for formatTime utility
docs: update AGENTS.md with testing rules
chore: bump @tauri-apps/api to 2.5.0
```

### PR Requirements

- [ ] All tests pass (`npm test`, `cargo test --workspace`)
- [ ] Lint passes (`npm run lint`, `cargo clippy`)
- [ ] Type check passes (`tsc --noEmit`)
- [ ] Agent sidecar builds (`cd agent-sidecar && npm run build`)
- [ ] Security scan passes (npm audit, cargo audit)
- [ ] PR description explains what and why

---

## 6. CI/CD

### Pull Request Checks (`.github/workflows/ci.yml`)

Triggers: `push` to `main`, `pull_request`

Jobs:
1. **Frontend Lint & Test**
   - `npm ci`
   - `npm run lint`
   - `npm run typecheck`
   - `npm test`
   - `npm run build:frontend`

2. **Agent Sidecar Lint & Build**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zosmaai/zosma-cowork](https://github.com/zosmaai/zosma-cowork) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
