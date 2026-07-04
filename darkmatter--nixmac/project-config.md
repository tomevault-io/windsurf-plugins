---
trigger: always_on
description: **nixmac** is a native macOS application (Tauri 2 + Rust backend, React 19 frontend) that puts an AI agent in front of a [nix-darwin](https://github.com/LnL7/nix-darwin) configuration. Users describe what they want in plain English and the app edits their Nix config files, builds the system, and applies it — including one-click rollback via git history.
---

# Copilot Cloud Agent Instructions

## What this repository is

**nixmac** is a native macOS application (Tauri 2 + Rust backend, React 19 frontend) that puts an AI agent in front of a [nix-darwin](https://github.com/LnL7/nix-darwin) configuration. Users describe what they want in plain English and the app edits their Nix config files, builds the system, and applies it — including one-click rollback via git history.

## Organization-wide agent guidance

- Organization-wide Copilot instructions are maintained in the `darkmatter/skills` repository.
- When reviewing pull requests for this repository, also apply and follow the PR review guidelines documented there.

## Repository layout

```
nixmac/
├── apps/native/          # The main deliverable: Tauri desktop app
│   ├── src/              # React/TypeScript frontend (Vite)
│   │   ├── components/widget/   # UI widgets (badges, controls, feedback, history,
│   │   │                        #  layout, notifications, overlays, promptinput,
│   │   │                        #  settings, steps)
│   │   ├── hooks/        # React hooks (use-evolve.ts, use-apply.ts, …)
│   │   ├── ipc/          # Tauri IPC bindings (api.ts, sqlite.ts, types.ts)
│   │   ├── stores/       # Zustand state (widget-store.ts)
│   │   └── stories/      # Storybook stories
│   └── src-tauri/        # Rust backend
│       └── src/
│           ├── main.rs           # App entry point; declares top-level modules only
│           ├── ai/               # ChatCompletionProvider trait + provider impls
│           │   └── providers/    # openai.rs, ollama.rs, cli.rs
│           ├── evolve/           # The AI evolution loop (tool use, file edits, git)
│           │   ├── mod.rs        # Core agent loop
│           │   ├── tools.rs      # Tool definitions (think/read_file/edit_file/…)
│           │   ├── file_ops.rs   # Path-safe file helpers (join_in_dir, resolve_*)
│           │   ├── edit_nix_file.rs  # Semantic Nix AST editing (rnix/rowan)
│           │   └── …
│           ├── rebuild/          # darwin-rebuild build/apply/rollback wrappers
│           ├── summarize/        # AI summarization pipeline
│           ├── commands/         # Tauri command handlers
│           ├── shared_types/     # Types shared between Rust and TypeScript via specta
│           ├── storage/          # Tauri store + keyring credential storage
│           ├── git/              # Git operations (exec, changes_from_diff)
│           ├── state/            # App state (build state, watcher, evolve state)
│           └── …
├── packages/ui/          # Shared Radix UI + Tailwind component library
├── nix/                  # devenv modules and Nix helper files
└── ops/                  # Release scripts (scripts/) and SOPS-encrypted secrets (secrets/)
```

## Tech stack

| Layer | Technologies |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Rust backend | Tauri 2, tokio, serde/serde_json, anyhow, thiserror, rusqlite + rusqlite_migration, specta (type export), rnix + rowan (Nix AST), clap (CLI), async-openai, tiktoken-rs |
| TypeScript frontend | React 19, Vite 7, Zustand, Radix UI, TailwindCSS 3, Monaco Editor, Shiki, Sonner, motion |
| Package manager | **Bun** (1.3.x) — use `bun install`, never `npm install` or `yarn` |
| Linting | **oxlint** (TS/JS), **biome** (formatting) |
| Testing | Vitest (unit + Storybook browser tests), Playwright (e2e web), WebdriverIO (e2e Tauri app) |
| Build system | `bun run desktop:build` (Tauri) wraps `cargo build` + Vite |
| CI | GitHub Actions — `.github/workflows/build.yaml` runs on `macos-latest` |
| Secrets | SOPS + age (`ops/secrets/secrets.sops.json`) — never commit plaintext secrets |

## ⚠️ macOS-only constraints for the cloud agent

nixmac targets macOS exclusively. The cloud agent runs on Ubuntu Linux; keep the following in mind:

- **The app cannot be fully built on Linux.** `tauri build` / `bun run desktop:build` requires macOS (Cocoa APIs, Apple signing). Do not attempt a production build in the agent environment.
- **Most Rust unit tests can run on Linux** via `cargo test --manifest-path apps/native/src-tauri/Cargo.toml`. Tests that invoke `darwin-rebuild` or macOS system APIs are guarded by `#[cfg(target_os = "macos")]` or the `e2e_mock_system` flag and will be skipped.
- **Frontend-only tests work fine** — `bun run test:unit` (Vitest/jsdom) runs on Linux.
- `devenv up` and `nix` commands require a Nix installation; do not rely on them in the agent.

## Building and testing (what works on Linux)

```bash
# Install JS/TS dependencies
bun install

# Rust unit tests (no macOS SDK required for most)
cargo test --manifest-path apps/native/src-tauri/Cargo.toml

# TypeScript unit tests
cd apps/native && bun run test:unit

# Storybook component tests (needs Playwright + Chromium installed)
cd apps/native && bun run test:storybook

# TS/JS lint

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [darkmatter/nixmac](https://github.com/darkmatter/nixmac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
