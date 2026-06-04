---
trigger: always_on
description: > Your AI remembers every conversation you've had.
---

# CLAUDE.md — Minutes

> Your AI remembers every conversation you've had.

## Project Overview

**Minutes** — open-source, privacy-first conversation memory layer for AI assistants. Captures any audio (meetings, voice memos, brain dumps), transcribes locally with whisper.cpp or parakeet.cpp, diarizes speakers, and outputs searchable markdown with structured action items and decisions. Built with Rust + Tauri v2 + Node.js (MCP).

**Four input modes, one pipeline:**
- **Live recording**: `minutes record` / `minutes stop` — for meetings, calls, conversations
- **Live transcript**: `minutes live` / `minutes stop` — real-time transcription with delta reads for AI coaching mid-meeting
- **Notetaking**: `minutes note "important point"` — timestamped annotations during recording
- **Folder watcher**: `minutes watch` — auto-processes voice memos from iPhone/iCloud

## Quick Start

```bash
cd ~/Sites/minutes
cargo build                          # Build Rust workspace
cargo test -p minutes-core --no-default-features  # Fast tests (no whisper model)
cargo run --bin minutes -- setup --model tiny      # Download whisper model
cargo run --bin minutes -- setup --diarization     # Download speaker diarization models (~34MB)
cargo run --bin minutes -- record    # Start recording
cargo run --bin minutes -- stop      # Stop and process
```

## Full Build (CLI + Tauri App)

```bash
./scripts/build.sh                   # Builds everything and installs CLI
./scripts/build.sh --install         # Same + copies .app to /Applications
./scripts/install-dev-app.sh         # Canonical signed dev app install to ~/Applications/Minutes Dev.app
# Or manually:
export CXXFLAGS="-I$(xcrun --show-sdk-path)/usr/include/c++/v1"
cargo build --release -p minutes-cli           # CLI binary
cargo tauri build --bundles app                # Tauri .app bundle
cp target/release/minutes ~/.local/bin/minutes # Install CLI
open target/release/bundle/macos/Minutes.app   # Launch app
```

**Hard rule for macOS desktop packaging and dogfooding:**

- If the work touches TCC-sensitive features, do **not** keep replacing `/Applications/Minutes.app` with local rebuilds.
- Use `./scripts/install-dev-app.sh` and test `~/Applications/Minutes Dev.app`.
- If a stable local codesigning identity exists, export `MINUTES_DEV_SIGNING_IDENTITY` before running the script.
- On this machine, the preferred identity is:
  - `Developer ID Application: Mathieu Silverstein (63TMLKT8HN)`
- Example:

```bash
export MINUTES_DEV_SIGNING_IDENTITY="Developer ID Application: Mathieu Silverstein (63TMLKT8HN)"
./scripts/install-dev-app.sh
```

**IMPORTANT**: After any code change, you must rebuild ALL affected targets:
- CLI changes: `cargo build --release -p minutes-cli && cp target/release/minutes ~/.local/bin/minutes`
- Tauri changes: `cargo tauri build --bundles app` then relaunch the appropriate app bundle
- TCC-sensitive desktop work (hotkeys, Screen Recording, Input Monitoring, Accessibility): `./scripts/install-dev-app.sh`
- MCP server changes: `cd crates/mcp && npm run build` (compiles TS server + builds UI, then restart MCP client sessions)
- MCP App UI only: `cd crates/mcp && npm run build:ui` (rebuild just the dashboard HTML)
- All Rust + app: `./scripts/build.sh` (add `--install` to copy .app to /Applications)
- **Don't forget the MCP server** — it's TypeScript, not Rust. `./scripts/build.sh` does NOT rebuild it. Always run `cd crates/mcp && npm run build` after touching `crates/mcp/src/index.ts` or `crates/mcp/ui/`.

## Desktop Identity Rules

For macOS permission-sensitive development, there are now two distinct desktop app identities:

- Production app:
  - name: `Minutes.app`
  - bundle id: `com.useminutes.desktop`
  - canonical install path: `/Applications/Minutes.app`
- Development app:
  - name: `Minutes Dev.app`
  - bundle id: `com.useminutes.desktop.dev`
  - canonical install path: `~/Applications/Minutes Dev.app`

Use the dev app for any work involving:

- dictation hotkeys / Input Monitoring
- Screen Recording prompts
- AppleScript / Accessibility automation
- any repeated TCC permission prompt investigation

Do not trust results from:

- `./Minutes.app`
- raw `target/debug/minutes-app`
- raw `target/release/minutes-app`
- repo-local bundle outputs launched directly from `target/`

Those identities are not stable enough for TCC debugging.

Native hotkey sanity check:

```bash
./scripts/diagnose-desktop-hotkey.sh "$HOME/Applications/Minutes Dev.app"
```

See [docs/DESKTOP-DEVELOPMENT.md](/Users/silverbook/Sites/minutes/docs/DESKTOP-DEVELOPMENT.md) for the full workflow.

For dictation shortcut work:

- prioritize the `Standard shortcut (recommended)` path first
- treat the raw-key `Caps Lock` / `fn` path as advanced and permission-heavy
- do not call the raw-key path “done” just because the monitor is active; require visible feedback or logged event delivery

### Open-source contributor note

This repo is public, so local scripts must not assume the maintainer's Apple
certificate or local notarization credentials.

- `./scripts/install-dev-app.sh` works without Apple credentials by falling
  back to ad-hoc signing
- for more stable TCC-sensitive testing, contributors can export
  `MINUTES_DEV_SIGNING_IDENTITY` to any consistent local codesigning identity

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [silverstein/minutes](https://github.com/silverstein/minutes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
