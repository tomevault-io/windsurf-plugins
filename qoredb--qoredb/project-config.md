---
trigger: always_on
description: Modern desktop database client built with Tauri 2 + React 19 + Rust.
---

# QoreDB

Modern desktop database client built with Tauri 2 + React 19 + Rust.
A lightweight, fast alternative to DBeaver/pgAdmin for developers.

## Collaboration principles (read first)

These principles take precedence over speed. For a trivial task, use your judgment.

### 1. Think before coding

**Don't assume. Don't hide confusion. Surface the trade-offs.**

Before implementing:

- State your assumptions explicitly. When in doubt, ask.
- If several interpretations are possible, present them — don't choose silently.
- If a simpler approach exists, say so. Push for it when it's warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity first

**The minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No unrequested "flexibility" or "configurability".
- No error handling for impossible scenarios.
- If you write 200 lines and 50 would do, rewrite.

Ask yourself: "Would a senior engineer say this is over-engineered?" If so, simplify.

### 3. Surgical changes

**Touch only what's necessary. Clean up only your own mess.**

When editing existing code:

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor what isn't broken.
- Respect the existing style, even if you'd do it differently.
- If you spot unrelated dead code, flag it — don't delete it.

When your changes create orphans:

- Remove the imports/variables/functions that YOUR changes made unused.
- Don't delete pre-existing dead code unless explicitly asked.

The test: every changed line must trace directly back to the user's request.

### 4. Goal-driven execution

**Define success criteria. Iterate until verified.**

Turn tasks into verifiable goals:

- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Make sure the tests pass before and after"

For multi-step tasks, state a brief plan:

```text
1. [Step] → verification: [check]
2. [Step] → verification: [check]
3. [Step] → verification: [check]
```

Strong success criteria let you iterate autonomously. Weak criteria ("make it work") require constant clarification.

## Tech stack

| Layer    | Technologies                                         |
| -------- | ---------------------------------------------------- |
| Frontend | React 19, TypeScript, Vite 7, Tailwind 4, CodeMirror |
| Backend  | Rust (edition 2024), Tauri 2, SQLx, tokio            |
| Database | PostgreSQL, MySQL, MongoDB, SQLite                   |

## Project structure

```
src/                    # React/TypeScript frontend
├── components/         # UI components (Browser/, Query/, Results/, ui/)
├── hooks/              # React hooks (useTabs, useTheme, useKeyboardShortcuts)
├── lib/                # Tauri bindings, utilities, types
└── locales/            # i18n translations (en.json, fr.json)

src-tauri/              # Rust backend (Cargo workspace)
├── src/                # Tauri binary crate
│   ├── commands/       # Tauri handlers (query, mutation, export, vault)
│   └── engine/         # Glue to the engine crates
└── crates/             # Workspace crates
    ├── qore-core/      # Engine abstraction (traits.rs, types, registry, error)
    ├── qore-drivers/   # Database drivers + session manager
    ├── qore-query/     # Query AST, compiler, dialects
    ├── qore-sql/       # SQL generation, safety, connection URLs
    ├── qore-service/   # Vault, governance/policy, service context
    └── qore-{cli,mcp,server}/  # Entry-point binaries

doc/                    # Detailed documentation
├── audits/             # Security & compliance audits
├── internals/          # Internal architecture
├── private/            # Open-core notes (internal)
├── release/            # Release process & events
├── rules/              # UI/design standards & features
├── security/           # Threat model, policies
├── tests/              # Testing constraints
└── todo/               # Roadmap & upcoming specs
```

## Essential commands

```bash
pnpm install            # Install dependencies
pnpm tauri dev          # Run the app in dev (hot reload)
pnpm lint:fix           # Lint + auto-fix
pnpm format:write       # Format the code
pnpm test               # Rust tests (cargo test)
pnpm tauri build        # Production build
```

Docker for test databases: `docker-compose up -d`

## Key architecture

**Frontend → Backend**: Calls go through `src/lib/tauri.ts`, which exposes typed bindings to the Rust commands.
**Database drivers**: Each driver implements the `DataEngine` trait (`src-tauri/crates/qore-core/src/traits.rs`), lives in `qore-drivers`, and is registered in the `DriverRegistry` (qore-core) from `qore-service/src/context.rs`.
**Security**: Encrypted vault (Argon2), SQL validation before execution (`qore-sql/src/safety.rs`), sandbox mode.

## Conventions

- Reusable UI components in `src/components/ui/` (based on shadcn/Radix)
- Custom hooks prefixed with `use*` in `src/hooks/`
- Tauri commands in `src-tauri/src/commands/`, exports in `lib.rs`
- Rust errors: custom types in `engine/error.rs`, propagation with `?`

## Open Core licensing (important)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [QoreDB/QoreDB](https://github.com/QoreDB/QoreDB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
