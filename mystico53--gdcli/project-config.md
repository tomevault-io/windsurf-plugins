---
trigger: always_on
description: Agent-friendly CLI for Godot 4. Provides structured, machine-readable output for linting GDScript, running projects headlessly, creating/editing scenes and scripts, looking up Godot API docs, managing UID references, and serving all commands over MCP.
---

# gdcli — Agent Developer Guide

Agent-friendly CLI for Godot 4. Provides structured, machine-readable output for linting GDScript, running projects headlessly, creating/editing scenes and scripts, looking up Godot API docs, managing UID references, and serving all commands over MCP.

## Project Overview

gdcli wraps a **stock Godot 4 binary** — no patched build needed. The CLI discovers the Godot binary dynamically (env var → PATH → common install paths) and probes its version at runtime. Error parsing uses Godot's standard `SCRIPT ERROR:` output format.

All commands emit a JSON envelope `{ ok, command, data, error }` when stdout is not a TTY or when `--json` is passed. This makes every command directly consumable by AI agents without extra parsing.

## Build & Test

```bash
# Build
cargo build

# Run tests
cargo test

# Lint with clippy
cargo clippy -- -D warnings

# Run a specific command
cargo run -- doctor
cargo run -- script lint --file path/to/script.gd
cargo run -- run --timeout 15
cargo run -- project info
cargo run -- scene list
cargo run -- scene validate path/to/scene.tscn
cargo run -- scene create path/to/new.tscn --root-type Node2D
cargo run -- scene edit path/to/scene.tscn --set Player::speed=200
cargo run -- node add scene.tscn Sprite2D MySprite
cargo run -- node remove scene.tscn MySprite
cargo run -- script create path/to/script.gd --extends Node --methods _ready
cargo run -- uid fix --dry-run
cargo run -- docs Node2D
cargo run -- docs --build
```

## Architecture

gdcli has a two-layer design:

| Layer | Commands | How it works |
|---|---|---|
| **Filesystem** | `project info`, `scene list/validate/create/edit`, `node add/remove`, `uid fix`, `script create`, `docs` | Pure file I/O — parses `.godot`, `.tscn`, `.tres`, `.uid`, and XML files directly. Instant, no Godot needed. |
| **Godot subprocess** | `doctor`, `script lint`, `run`, `docs --build` | Spawns the Godot binary with `--headless` and captures stdout/stderr. Works with stock Godot 4. |
| **MCP server** | `mcp` | JSON-RPC 2.0 over stdio — exposes all commands as MCP tools for AI clients. |

### JSON Envelope

Every command outputs the same envelope shape:

```json
{
  "ok": true,
  "command": "script lint",
  "data": { ... },
  "error": null
}
```

- `ok`: `true` if no errors/issues found, `false` otherwise
- `command`: the subcommand name
- `data`: command-specific payload (see each command module's report struct)
- `error`: human-readable error string, or `null`

JSON mode activates automatically when stdout is not a TTY (pipe, redirect, agent invocation). Force it with `--json`.

## Source Layout

| File | Purpose |
|---|---|
| `src/main.rs` | CLI entry point. Defines clap commands/subcommands, routes to handlers. Splits filesystem commands (no Godot needed) from subprocess commands. |
| `src/runner.rs` | Spawns Godot as a subprocess. `run()` adds `--headless`; `run_raw()` runs with exact args. Drains stdout/stderr in background threads to prevent pipe deadlocks. Handles timeouts via `wait-timeout`. |
| `src/godot_finder.rs` | Discovers Godot binary (`GODOT_PATH` → `which` → common paths). Probes `--version`. Prefers `.console.exe` on Windows. |
| `src/output.rs` | `JsonEnvelope<T>` struct, `use_json()` TTY detection, `emit_json()`, colored TTY helpers (`print_check`, `print_header`, `print_error`). |
| `src/errors.rs` | Parses Godot error output. `parse_script_errors()` handles `SCRIPT ERROR:` blocks from Godot's standard output. Filters noise errors (`.godot/` internal, global script cache). |
| `src/scene_parser.rs` | Parses `.tscn` files into `ParsedScene` (header, ext_resources, sub_resources, nodes, connections). Also provides `find_scene_files()` for recursive `.tscn` discovery. |
| `src/commands/mod.rs` | Module declarations for all command handlers. |
| `src/commands/doctor.rs` | `gdcli doctor` — checks Godot binary, `project.godot` presence, `.gd` file count. |
| `src/commands/script.rs` | `gdcli script lint` — single-file lint via `--check-only`, project-wide lint via `--headless --quit`. Both parse `SCRIPT ERROR:` blocks. |
| `src/commands/run.rs` | `gdcli run` — runs project headlessly with timeout. Reports exit code, stdout, stderr, parsed errors, timeout status. |
| `src/commands/project.rs` | `gdcli project info` — parses `project.godot` for name, main scene, autoloads, script/scene counts. |
| `src/commands/scene.rs` | `gdcli scene list` / `gdcli scene validate` — lists scenes with node counts, validates ext_resource paths exist on disk. |
| `src/commands/uid.rs` | `gdcli uid fix` — scans `.uid` files to build UID→path map, finds stale path refs in `.tscn`/`.tres`, fixes them. `--dry-run` supported. |
| `src/commands/docs.rs` | `gdcli docs` — looks up Godot API docs from cached XML class reference. `--build` runs `godot --doctool` to generate the cache. |
| `src/commands/node.rs` | `gdcli node add/remove` — adds or removes nodes in `.tscn` files. Handles ext_resource management, parent paths, script attachment, and property setting. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mystico53/gdcli](https://github.com/mystico53/gdcli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
