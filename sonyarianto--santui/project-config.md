---
trigger: always_on
description: Automates the whole flow: pre-flight check (every version-bearing file must be at the
---

# Santui — Agent Guide

## Build & Test

```bash
./scripts/check.sh check    # fast compile check — host + core + stable plugins only
./scripts/check.sh clippy   # lint (same fast set)
./scripts/check.sh test     # tests (core crates + stable plugin tests)
./scripts/check.sh fmt      # formatting check
cargo fmt                   # auto-format

cargo build -p santui       # build just the host app (add specific -p flags for more)
cargo check --workspace     # SLOW — all ~127 crates incl. experimental plugins (CI's job)
cargo clippy --workspace -- -D warnings  # SLOW — only needed when touching experimental plugins
cargo clippy --workspace --all-targets -- -D warnings  # SLOW — ONLY this flag lints plugin #[cfg(test)] code; CI gate
cargo test --workspace      # SLOW — compiles every plugin binary; prefer `./scripts/check.sh test`
```

**Default verification = `scripts/check.sh`** (host + core + builtins + stable plugins from `plugins-manifest.json`; stable set is generated via `santui-dev-setup list-ids`). Full-workspace verification runs in CI on every push — you only need `--workspace` locally when you're working on an experimental plugin. lefthook pre-commit runs `cargo fmt --check` + `./scripts/check.sh clippy` automatically. Install hooks: `lefthook install`.

**Important**: `check.sh clippy` and the pre-commit hook lint non-test code only. Plugin `#[cfg(test)]` code (and any `#[cfg(test)]`-gated items like test modules) is compiled and linted **only** by CI's `cargo clippy --workspace --all-targets -- -D warnings`. When you edit a plugin's tests or reorder code around a `mod tests` block, run that exact command locally — a dropped `#[cfg(test)]` attribute or a bad test-only pattern compiles fine in the fast set but fails CI.

Run: `cargo build -p santui && cargo run -p santui` or `.\target\debug\santui.exe`

Server: `cargo run -p santui-server`

Dev mode (plugin registry + native deps):
  - Windows: `.\scripts\dev-setup.ps1 ; $env:SANTUI_DEV=1; cargo run -p santui`
  - macOS/Linux: `./scripts/dev-setup.sh && SANTUI_DEV=1 cargo run -p santui`

Fast dev (skip building plugins entirely — just run the app):
  - `./scripts/dev-setup.sh --no-build && SANTUI_DEV=1 cargo run -p santui`

Dev-setup now builds only the host binary + builtins (`santui`, `santui-dev-setup`, `santui-registry-plugin`) plus plugins with `"status": "stable"` in `plugins-manifest.json`. Other experimental plugins are compiled only when running `cargo build --workspace` explicitly.

Watch: `cargo watch -x "run -p santui"`

## Workspace

```
crates/
├── core/          — framework: App, Plugin trait, event loop, palette, sync client
├── ipc/           — IPC protocol types + host (`IpcPluginHost`) plugin runner
├── auth/          — GitHub OAuth + auth handle/client
├── db/            — central SQLite database for per-user plugin data
├── registry/      — plugin registry: manifest fetch, install, config
├── server/        — optional self-hosted sync server (axum + SQLite + JWT)
├── plugins/           — first-party plugins (see plugins-manifest.json for full list)
│   ├── radio-stream-player/   — radio plugin
│   │   └── scraper/           — scrape radio stations into DB (--db-path, --prune, --help)
│   ├── registry/             — plugin registry UI plugin
│   └── ... (more plugin directories)
├── app/           — binary entry point (main.rs)
└── website/       — VitePress docs site
```

## Key Conventions

- Rust edition 2024, no nightly
- `ratatui` for rendering; `Theme` semantic colors over hardcoded `Color::*`
- `impl Default` for any type with a `new()` constructor
- `cargo fmt` before commit; clippy must pass with `-D warnings` (enforced by lefthook pre-commit)
- Commit messages must be in English
- **Refactoring / non-trivial changes**: work on a feature branch, push for review, then merge to `main`
- **Don't push on every commit** — only push when explicitly asked or when the branch is ready for review/merging
- **Semantic correctness**: before/after each edit, read the full surrounding function to ensure variable names, types, and logic still make sense. The compiler catches type errors but NOT wrong variable names (e.g. `name` vs `id`) or wrong control flow (e.g. `return` vs `continue`). Re-read the diff yourself before staging.
- **Verify claims against code**: never assert facts about the codebase (which plugins persist data, how many tests exist, what a feature does) from memory — memory drifts between sessions. Grep/read the code first, then answer. A confident-sounding but wrong claim (e.g. "radio doesn't persist favorites" when it does via `DbSet "favorites"`) wastes the user's time and erodes trust. If asked to compare features across crates/plugins, verify every side of the comparison.
- **Structural vs semantic filtering**: When filtering a collection (plugins, messages, events), prefer semantic criteria (e.g. "was installed via registry") over structural ones (e.g. "has a binary path"). Built-in plugins can share the same structure as registry-installed ones. A wrong filter compiles fine but causes subtle breakage at runtime (e.g. killing the registry plugin on startup). Add a dedicated tracking field rather than reusing an existing one for a different purpose.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sonyarianto/santui](https://github.com/sonyarianto/santui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
