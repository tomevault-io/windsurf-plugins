---
trigger: always_on
description: This file provides guidance to AI agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI agents when working with code in this repository.

## Project Overview

All slop must die. Airlock is where every git push turns into a slop-free PR.

Airlock is a local-first CI system that transforms messy AI-generated code into clean, reviewable pull requests. It intercepts `git push` operations through a local bare repo "gate", runs a transformation pipeline, and lets users review changes before forwarding to upstream.

## Build Commands

```bash
# Using Makefile (recommended)
make dev              # Start desktop app in dev mode (hot reload)
make build            # Build frontend + all Rust crates
make test             # Run all tests
make check            # Run clippy, format checks, and frontend lint
make fmt              # Format all code (Rust + frontend)

# Run `make` or `make help` to see all available commands.


# Direct cargo commands (only if make commands don't fit your needs)
cargo build              # Build all Rust crates (frontend must be pre-built)
cargo test               # Run all tests
cargo fmt                # Format code
cargo clippy             # Run lints
cargo run --bin airlock  # Run CLI (launches GUI if no args)
cargo run --bin airlockd # Run daemon
```

Never run multiple make or cargo commands in parallel. It will cause lock contention. Run one command at a time, and
wait for it to finish before starting another.

Never run cargo tests in parallel (e.g. `--test-threads=N` where N > 1). Tests share database and filesystem
resources that cause deadlocks under parallel execution. Always use the default single-threaded test runner.

## Desktop App Development

The desktop app uses Tauri (Rust) + React (TypeScript). We use shadcn for UI components (look up context7 for docs when needed).

### Development Mode

When making frontend changes in the desktop app, you should run in development mode with mock data and hot reload:

```bash
cd crates/airlock-app && npm run dev
```

After making UI changes, you need to **always** use chrome-devtools to test the UI, otherwise you don't know if it works.

### Production Build

```bash
make build
# This builds frontend assets first, then compiles Rust
```

**If you see a white screen:** The frontend hasn't been built. Run `make build` or `make dev`.

### Color Guidelines

Use semantic colors instead of hard-coded Tailwind colors for theming support:

| Purpose                  | Use                                       | Avoid                          |
| ------------------------ | ----------------------------------------- | ------------------------------ |
| Success/passed states    | `text-success`, `bg-success`              | `text-green-*`, `bg-green-*`   |
| Warning/pending states   | `text-warning`, `bg-warning`              | `text-yellow-*`, `bg-yellow-*` |
| Error/failed states      | `text-danger`, `bg-danger`                | `text-red-*`, `bg-red-*`       |
| Signal/accent states     | `text-signal`, `border-signal`            | `text-blue-*`, `bg-blue-*`     |
| Terminal/log backgrounds | `bg-terminal`, `text-terminal-foreground` | `bg-zinc-*`, `text-zinc-*`     |

`destructive` is an alias for `danger`; both work. Semantic colors are defined in `packages/design-system/src/styles/tokens.css` and the Tailwind preset at `packages/design-system/src/tailwind/preset.js`.

## Pre-commit Checklist

Before committing, always run `make check` to catch CI failures early. This runs:

- `cargo fmt --check` - Rust formatting
- `cargo clippy` - Rust lints
- `npm run lint` - ESLint (catches unused imports, etc.)
- `npm run format:check` - Prettier formatting

If lint fails, run `make fmt` to auto-fix formatting, and manually fix any lint errors (e.g. remove unused imports).

## Rules

Before implementing a fix, understand the full architecture constraints — especially shared resources (e.g., worktree pool concurrency), plugin lifecycle ordering (e.g., window-state plugin restores after setup), and pipeline customizability (steps may not always run). Ask clarifying questions if unsure rather than assuming.

When referencing external library APIs (Clerk, OpenAI SDK, etc.), verify against official documentation rather than relying on training data. APIs change between major versions. If unsure, say so and look up the docs.

## Debugging

When debugging production issues, systematically enumerate and test hypotheses rather than chasing the first plausible theory. Check environment variables, build configs (eas.json, .env), and server-side overrides before assuming code-level bugs.

## Testing Requirements

After implementing new features, always write:

1. **Unit tests** - Test individual functions and logic in isolation
2. **Integration/E2E tests** - Test the full flow with real git repos and database

### Test Performance

Use the smallest possible timeouts and delays in tests. If a test needs a timeout to trigger, use 1ms — not
100ms, not 1s. Every unnecessary millisecond adds up across the test suite.

### E2E Test Principles

E2E tests must reflect **end-user-visible behavior**. Do things the way a real user would:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [airlock-hq/airlock](https://github.com/airlock-hq/airlock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
