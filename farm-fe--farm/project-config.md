---
trigger: always_on
description: This file provides project-level context for AI coding assistants working in
---

# Farm — AI Agent Instructions

This file provides project-level context for AI coding assistants working in
this repository.

## Project Overview

**Farm** is an extremely fast, Vite-compatible web build tool written in Rust.
It is a monorepo that contains the core compiler (Rust), official JS/Rust
plugins, runtime packages, a CLI, a documentation website, and examples.

- Website: <https://farmfe.org>
- npm: `@farmfe/core`
- License: MIT

---

## Repository Structure

```
farm/
├── crates/            Rust workspace — compiler core and most Rust plugins
│   ├── compiler/      Main compilation engine
│   ├── core/          Rust/Node bridge (napi-rs)
│   ├── plugin_*/      Official Rust plugins (CSS, HTML, JSON, resolve …)
│   └── node/          Node.js native bindings
├── packages/          TypeScript packages (runtime, CLI, etc.)
├── js-plugins/        Official JavaScript plugins (postcss, less, sass, svgr, dts, visualizer, tailwindcss, electron)
├── rust-plugins/      Community-maintained Rust plugins
├── examples/          Standalone example projects (each is its own workspace package)
├── website/           Docusaurus documentation site
├── scripts/           Build, release and tooling scripts
└── e2e/               Playwright E2E tests
```

---

## Key Commands

| Command                | Purpose                                                     |
| ---------------------- | ----------------------------------------------------------- |
| `pnpm bootstrap`       | First-time setup: install deps + build core packages        |
| `pnpm run ready`       | Full CI gate — install, clean, build, all tests (see below) |
| `pnpm run test`        | Unit tests (vitest)                                         |
| `cargo test`           | Rust unit tests                                             |
| `pnpm run test-e2e`    | Playwright E2E tests                                        |
| `node scripts/test-examples.mjs --skip-build-js-plugins --example <name>` | Build and validate one example only.                        |
| `pnpm run check`       | Biome lint + format                                         |
| `cargo clippy`         | Rust linter                                                 |
| `pnpm run spell-check` | cspell across all files                                     |

### Example Test Argument Mapping

- `--example <name>` → single-example mode (build only that example)
- `--from <name>` / `--start-from <name>` → build from that example to the end of the sorted example list
- `--skip-build-js-plugins` → skip JS plugin build stage
- Full guide: `docs/experiences/example-testing-guide.md`

### Ready Gate (full CI parity)

`pnpm run ready` runs in order:

1. `pnpm install`
2. `node ./scripts/clean.mjs`
3. `npx cspell "**" --gitignore`
4. Build core / JS & Rust plugins / CLI (`runTaskQueue`)
5. `cargo check --all --all-targets`
6. `cargo clippy`
7. `pnpm run --filter "@farmfe/*" type-check`
8. `pnpm run test`
9. `cargo test -j <N>`
10. Build core CJS
11. Build examples
12. `pnpm run test-e2e`

---

## Tech Stack & Conventions

### Rust

- Rust edition 2021, toolchain pinned in `rust-toolchain.toml`
- Formatting via `rustfmt.toml`; lint with `cargo clippy`
- Crates share workspace dependencies declared in the root `Cargo.toml`

### TypeScript / JavaScript

- **Formatter + linter:** Biome (`biome.json`). Run `pnpm run check` before committing.
- **Package manager:** pnpm v9+ with workspaces (`pnpm-workspace.yaml`)
- **Module format:** ESM in source; dual CJS+ESM output for published packages
- **Type checking:** `tsc` per-package via `type-check` script
- All TS config files extend `tsconfig.base.json`

### Git

- Commit messages follow Conventional Commits (`feat:`, `fix:`, `chore:`, etc.)
- Changesets managed via `@changesets/cli` — run `npx changeset` when bumping a package version
- PR titles are linted by `lint-pr-title.yml` — must match Conventional Commits

---

## Coding Guidelines

- **Minimal changes.** Only touch files directly required by the task.
- **No new features** unless explicitly requested.
- **Security.** No use of `eval`, dynamic `require` with user input, or unsafe Rust `unwrap()` on user-facing paths.
- **Tests.** Add or update tests when fixing bugs or adding features.
- **Docs.** Update `website/docs/` when changing public API, config options, or plugin interfaces.

---

## Available AI Agents

| Agent                | Trigger                                                               | File                                       |
| -------------------- | --------------------------------------------------------------------- | ------------------------------------------ |
| **FarmFE Docs Sync** | After code changes that affect docs; finding doc/code discrepancies   | `.github/agents/farmfe-docs-sync.agent.md` |
| **Explore**          | Read-only codebase Q&A, researching architectures, tracing code paths | Built-in subagent                          |

---

## Available Skills (Slash Commands)

Skills are on-demand workflows invoked as slash commands in VS Code Copilot chat. They are discovered automatically from three directories:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [farm-fe/farm](https://github.com/farm-fe/farm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
