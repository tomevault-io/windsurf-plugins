---
trigger: always_on
description: > This file is the source of truth for AI agents working on this repo.
---

# AGENT.md — Excalidraw Preview for Zed

> This file is the source of truth for AI agents working on this repo.
> `CLAUDE.md` is a symlink to this file.

## Project Purpose

A Zed editor extension that previews `.excalidraw` files in a native WebView window (powered by `wry`). The preview live-reloads on file save. No browser tabs. No in-editor UI panes. Pure offline, near-zero-latency diagram preview.

Supports all three Excalidraw file formats: `.excalidraw` (JSON), `.excalidraw.svg`, `.excalidraw.png`.

See [`docs/PRD.md`](docs/PRD.md) for the full product requirements.
See [`refs/excalidraw-vscode/`](refs/excalidraw-vscode/) (git submodule) for the reference implementation this is adapted from.

---

## Repository Layout

```
excalidraw-zed-extension/
│
├── AGENT.md                        ← you are here (source of truth)
├── CLAUDE.md                       ← symlink → AGENT.md
├── docs/
│   └── PRD.md                      ← full product requirements doc
│
├── extension/                      ← Zed extension (Rust → WASM)
│   ├── Cargo.toml
│   ├── src/
│   │   └── lib.rs                  ← slash command, spawn binary, focus ping
│   └── extension.toml              ← Zed extension manifest
│
├── preview-binary/                 ← companion native binary
│   ├── Cargo.toml
│   ├── src/
│   │   └── main.rs                 ← CLI entry, all routes, file watcher, WebView (monolith)
│   ├── webview-src/                ← React + Vite source (npm project)
│   │   ├── package.json            ← @excalidraw/excalidraw ^0.18, react ^18, vite
│   │   ├── vite.config.ts          ← prod: assets only; dev: mock API plugin for any file
│   │   ├── index.html
│   │   └── src/
│   │       ├── main.tsx            ← fetch /config + /data → loadFromBlob → render, SSE
│   │       └── App.tsx             ← <Excalidraw> editor, Ctrl+S / auto-save, SSE reload
│   └── assets/                     ← Vite build output; committed; embedded at compile time
│       ├── index.html              ← served at GET /
│       └── assets/
│           ├── index-[hash].js     ← React + Excalidraw bundle
│           ├── index-[hash].css
│           └── *.woff2, *.wasm     ← Excalidraw runtime assets (GET /assets/*)
│
├── refs/
│   └── excalidraw-vscode/          ← git submodule: VS Code reference implementation
│
├── .claude/
│   └── skills/
│       └── zed-extension/
│           └── SKILL.md            ← Zed extension scaffolding skill
│
└── Cargo.toml                      ← workspace root
```

---

## Workspace Cargo.toml (root)

```toml
[workspace]
members = [
    "extension",
    "preview-binary",
]
resolver = "2"
```

---

## Component 1 — Zed Extension (`extension/`)

**Target:** `wasm32-wasip1`
**Crate type:** `cdylib`

### extension.toml

```toml
id = "excalidraw-preview"
name = "Excalidraw Preview"
version = "0.1.0"
schema_version = 1
authors = ["you"]
description = "Preview .excalidraw files in a native window"
repository = "https://github.com/you/excalidraw-zed-extension"

[slash_commands.preview-excalidraw]
description = "Open live preview for the active .excalidraw file"
requires_argument = false
```

### extension/src/lib.rs — responsibilities

1. Implement `zed_extension_api::Extension` trait.
2. Register `/preview-excalidraw` slash command.
3. On command run:
   - Get `worktree` + active file path via extension API.
   - Validate extension is `.excalidraw`, `.excalidraw.svg`, or `.excalidraw.png`.
   - Resolve path to companion binary (`excalidraw-preview`).
   - Spawn via `zed_extension_api::process::Command::new(binary).arg(file_path).spawn()`.
4. Track `HashMap<PathBuf, (u32, u16)>` — PID + port per file (in-memory, single process lifetime).
5. On re-invoke for same file: `GET http://127.0.0.1:{port}/focus` (HTTP ping via `zed::http_client_get`).

**Key constraint:** WASM extensions cannot open sockets or use `std::process`.
Use `zed_extension_api::process::Command` and `zed::http_client_get` only.

---

## Component 2 — Companion Binary (`preview-binary/`)

**Target:** native (x86_64/aarch64, macOS/Linux/Windows)

**Note:** all server, watcher, webview, and asset logic lives in a single `main.rs` (monolith); the AGENT.md originally described separate files that were never split out.

### CLI

```
excalidraw-preview <file-path> [--port <port>] [--auto-save] [--debug]
excalidraw-preview --lsp
excalidraw-preview --dev
excalidraw-preview --dev-server <url>
```

### Startup sequence

1. Parse args with `clap`.
2. If `--lsp`: run JSON-RPC LSP server loop (Zed language server integration).
3. If `--dev` / `--dev-server`: open WebView at the Vite dev server URL directly.
4. Otherwise: detect file format from extension → MIME type string.
5. Check lock file `$TMPDIR/excalidraw-{sha256(canonical_path)}.lock`.
   - If live (`GET /ping` succeeds) → send `GET /focus` and exit.
   - If stale → remove and continue.
6. Bind axum server on ephemeral port (or `--port`).
7. Write port to lock file.
8. Spawn file watcher thread (notify v6, 80 ms debounce → broadcast channel).
9. Open WebView window at `http://127.0.0.1:{port}`.
10. On window close: remove lock file, shut down server.

### HTTP Routes

| Route | Description |
|---|---|
| `GET /` | Serve embedded `index.html` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arindampradhan/excalidraw-zed-extension](https://github.com/arindampradhan/excalidraw-zed-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
