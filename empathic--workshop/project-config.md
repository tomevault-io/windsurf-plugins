---
trigger: always_on
description: When changing code, **update the associated docs in the same change**. Key files: `docs/architecture.md` (system design), `docs/web-terminal.md` (client terminal), `docs/configuration.md` (config), `packages/workshop_ui/CLAUDE.md` (frontend conventions), this file (build/architecture notes). Stale docs are worse than no docs.
---

# CLAUDE.md

## Documentation

When changing code, **update the associated docs in the same change**. Key files: `docs/architecture.md` (system design), `docs/web-terminal.md` (client terminal), `docs/configuration.md` (config), `packages/workshop_ui/CLAUDE.md` (frontend conventions), this file (build/architecture notes). Stale docs are worse than no docs.

## Build System

This is a monorepo with Bazel, Cargo, and TS/JS build systems. Everything must stay in sync.

### Crate Features

When adding a crate feature (e.g. `reqwest`'s `blocking`), update **both**:
1. `Cargo.toml` — the package's `[dependencies]` features list
2. `MODULE.bazel` — the corresponding `crate_index.spec()` features list

### Formatting

Always use `bazel run //tools/format` to format code. Do not run `rustfmt` directly.

### Git

All commits **must be GPG-signed**. Never use `--no-gpg-sign`. If signing agent
errors occur, ask the user to unlock the signing agent or fix the issue rather
than bypassing signing.

### Rust Edition 2024

All Rust code uses edition 2024. Cargo defaults to edition 2021 for `cargo check`/`cargo test`, so some edition 2024 errors only surface in Bazel. Known gotcha: `ref mut` in match/if-let patterns is disallowed when the default binding mode is already `ref mut` (e.g. matching on `&mut Option<T>` — use `Some(x)` not `Some(ref mut x)`).

### Build Commands

- `cargo check -p workshop` — quick compile check
- `cargo test -p workshop` — run unit tests for the server
- `cargo test -p <package>` — run unit tests for any workspace crate
- `bazel test //...` — full CI-equivalent (includes format check, edition 2024)
- `WORKSHOP_UI_PATH=packages/workshop_ui/build cargo build -p workshop_ui` — build embedded UI crate

### Desktop App (Tauri)

- `cargo check -p workshop_desktop` — quick compile check
- `cargo test -p workshop_desktop` — run unit tests
- `cd packages/workshop_desktop && cargo tauri dev --config tauri.dev.conf.json` — launch desktop app with embedded server (auto-starts Vite dev server)
- `bazel build //packages/workshop_desktop:macos_app` — build macOS `.app` bundle (debug)
- `bazel build --config=opt //packages/workshop_desktop:macos_app` — build optimized `.app` bundle

**Dev workflow** (single terminal): `cd packages/workshop_desktop && cargo tauri dev --config tauri.dev.conf.json` — the Tauri app starts an embedded server in-process, and Vite's dev proxy discovers it automatically via the `daemon.port` file. The `--config` flag merges `tauri.dev.conf.json` (devUrl + beforeDevCommand) into the base config. The base `tauri.conf.json` has no dev URL — production builds never reference external dev servers.

**Custom data directory**: `workshop_desktop --data-dir /path/to/data` (defaults to `~/.workshop`).

Note: `workshop_desktop` is in workspace `members` but NOT in `default-members` (requires Tauri system deps). The desktop app depends on the `workshop` library crate (with `embedded-ui` feature) — no separate daemon process.

### Frontend (SvelteKit)

- `cd packages/workshop_ui && pnpm install && pnpm build` — build the web UI
- `cd packages/workshop_ui && pnpm dev` — dev server with hot reload
- `cd packages/workshop_ui && pnpm test` — run Jest tests
- `cd packages/workshop_ui && pnpm format` — format TS/Svelte with Prettier (also runs via `bazel run //tools/format`)

## TUI Styling

The terminal theme is solarized. Hardcoded ANSI colors are invisible or clash:
- **Avoid**: `Color::DarkGray`, `Color::White`, `Color::Cyan`, `Color::Black`, `Color::Green`, `Color::Yellow`
- **Use modifiers**: `BOLD`, `DIM`, `REVERSED`, `ITALIC`, `UNDERLINED`
- **Exception**: `Color::Red` is safe (universally visible, use for errors)

## Architecture Notes

### Package Dependency Graph

```
workshop (lib: server core, config, handlers, WS | bin "work": CLI + TUI)
├── claude_convo      (conversation log reader)
├── pty_manager        (PTY lifecycle)
└── virtual_terminal   (screen buffer + viewport negotiation)

tty_wrapper            (standalone HTTP-controlled PTY — not depended on by workshop)
workshop_ui           (SvelteKit frontend — embedded via rust-embed feature flag)
workshop_desktop      (Tauri native desktop app — embeds workshop server in-process)
  └── workshop (lib, with embedded-ui feature)
```

### Daemon Lifecycle

One server per data directory, enforced by advisory file lock (`daemon.lock`). The lock uses `flock(2)` — automatically released on process crash.

- **`try_acquire_daemon_lock()`** — non-blocking exclusive lock attempt; returns `None` if another server holds it
- **`check_existing_server()`** — reads `daemon.pid`/`daemon.port`, verifies process alive via `kill(pid, 0)`, then health-checks `GET /health`
- **`release_daemon_files()`** — PID-aware cleanup: only deletes state files if `daemon.pid` matches current process
- **`DaemonLock`** — RAII guard; `Drop` calls `release_daemon_files()`, then releases the flock
- Both `work server` and `EmbeddedServer::start()` acquire the lock before initializing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [empathic/workshop](https://github.com/empathic/workshop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
