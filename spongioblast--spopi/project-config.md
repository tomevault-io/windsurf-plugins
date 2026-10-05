---
trigger: always_on
description: This file contains repository-wide development rules. The documentation map:
---

# SPOPI agent guide

This file contains repository-wide development rules. The documentation map:

| File | What it answers |
|---|---|
| [`docs/FEATURES.md`](docs/FEATURES.md) | What every feature does for the user, how to reach it, how to configure or turn it off, its limits, and which Pi piece it relies on |
| [`ARCHITECTURE.md`](ARCHITECTURE.md) | Rules the code must keep: transport, security and host boundaries, module ownership, each feature's invariants |
| [`docs/DESIGN.md`](docs/DESIGN.md) | Design tokens and UI primitives |
| [`docs/RPC_COVERAGE.md`](docs/RPC_COVERAGE.md) | Every Pi RPC command and event: handled, unused, or forbidden |
| [`docs/PI_BUMP.md`](docs/PI_BUMP.md) | Bumping the embedded Pi and the updater key |
| [`CHANGELOG.md`](CHANGELOG.md) | Short release notes, one section per version; `release.yml` uses the tag's section as the GitHub release text |
| [`README.md`](README.md) | The short pitch and how to run |

## Read first

- Before answering what SPOPI does or changing a feature, read its section in
  `docs/FEATURES.md`. Before changing how it works, read the applicable
  `ARCHITECTURE.md` section and its linked design documents. This applies to UI
  behavior, persistence, workspace I/O, and cross-process communication.
- Update `docs/FEATURES.md` in the same change when a user-visible behavior,
  label, shortcut, setting, default, file location, or limit changes, or a
  feature is added or removed. Keep its section format (Use, Configure, Limits,
  Pi, Code).
- For each user-visible change, add one short line to `CHANGELOG.md` under the
  next version (`## X.Y.Z`), in a **Features** or **Fixes** list.
- Update `ARCHITECTURE.md` when an implementation materially changes its
  architecture, invariants, lifecycle, security boundary, or validation
  contract. Changes to LAN access, cross-platform paths, or static serving also
  require the corresponding architecture update.

Tauri wraps the web UI. Rust starts a native `HostServer` plus a managed `pi --mode rpc` subprocess using the embedded pi binary shipped in `src-tauri/resources/pi/` (downloaded by `scripts/fetch-pi-binary.js` from pi-mono releases at the version pinned in `scripts/pi-version.json`). The WebView talks to the Rust host over `/v2/ws`; the host bridges runtime requests to Pi over stdio RPC.

```
SPOPI .app
  resources/
    public/                        (frontend)
    extensions/                    (bundled Pi extensions, wasm, verify skills)
    skills/spopi-customize/        (bundled skill)
    pi/<bun-compiled pi binary + assets>
  Rust HostServer + PiRuntime
    spawn pi --mode rpc --extension spopi-bridge.mjs …
    WebView  →  /v2/ws  →  HostServer  →  stdio RPC  →  pi
```

Runtime, data, auth, and extension UI traffic goes through the native host protocol (`/v2/ws`). The six Tauri commands in `src-tauri/src/commands.rs` are only for native window and workspace actions the WebView cannot do: pick a folder, relocate a missing project, open another project or session in a window, show a task notification, and retry a failed start. Do not add Tauri commands for Pi, files, git, or preferences.

### Goals

- Local desktop GUI: all projects and agents visible in one app
- Multi-project: each project has its own window, isolated working directory, session history, and running agent
- Multi-agent: spawn new agents per project; switch between sessions without leaving the app
- Native runtime protocol: browser frames are routed by Rust over `/v2/ws`, then forwarded to the managed Pi process over stdio RPC.
- Visualization: streaming chat, tool-call cards, thinking blocks, token/cost tracking per session
- Fully self-contained desktop app: zero dependency on the user's PATH / shell environment / globally installed pi

### Constraints

- Frontend: vanilla JS, no framework (`public/`)
- Backend: Rust (Tauri) owns process lifecycle, the HTTP/WebSocket host, routing, and host data APIs
- PI integration: always via embedded `pi --mode rpc` subprocess — never re-implement PI runtime logic
- Session history and working directory are isolated per project/port
- The embedded pi version is the source of truth: `pi --version` shown in the UI comes from `SPOPI_PI_VERSION` (set by Rust at spawn time, populated from `scripts/pi-version.json`). A user-installed pi on `$PATH` is irrelevant and never touched.
- User extensions under `~/.pi/agent/extensions/` and `<workspace>/.pi/extensions/` are still auto-loaded by the embedded pi (embedding doesn't disable user extensions).

### PI references

Docs ship inside the embedded pi runtime at `src-tauri/resources/pi/docs/` (populated by `bun run fetch:pi`; see "Bumping the embedded pi version" below). Prefer these repo-relative paths over any globally-installed `pi-coding-agent` — a global install may not exist on a given machine or may be a different version than the one pinned in `scripts/pi-version.json`.

- RPC protocol: `src-tauri/resources/pi/docs/rpc.md`
- SDK: `src-tauri/resources/pi/docs/sdk.md`
- Session format: `src-tauri/resources/pi/docs/session-format.md`
- JSON mode: `src-tauri/resources/pi/docs/json.md`

---

# Agent working notes

Conventions for any coding agent working in this directory.

## Agent skills

### Issue tracker


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spongioblast/spopi](https://github.com/spongioblast/spopi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
