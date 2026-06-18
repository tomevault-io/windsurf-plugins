---
trigger: always_on
description: macOS menubar app + CLI for running local LLMs via `mlx-lm` on Apple Silicon.
---

# ollmlx — Agent Instructions

## Project overview

macOS menubar app + CLI for running local LLMs via `mlx-lm` on Apple Silicon.

Three targets share one repository:
- **OllmlxCore** — shared framework, no UI dependencies
- **OllmlxApp** — menubar app (AppKit + SwiftUI)
- **ollmlx** — CLI (ArgumentParser, no AppKit)

The daemon (`ServerManager`) spawns and monitors `mlx_lm.server` as a child process. The CLI and menubar app communicate with the daemon exclusively via HTTP on `localhost:11435`. The public OpenAI-compatible API is exposed on `localhost:11434` via a transparent proxy. Ollama-compatible endpoints (`/api/tags`, `/api/version`, `/api/chat`, `/api/generate`) are also served on `:11434` so clients like Open WebUI can connect natively.

Full implementation plan: `ollmlx-implementation-plan.md`

---

## Build

```bash
# Open in Xcode 15+
open ollmlx.xcodeproj

# Bootstrap Python venv (run once before testing)
bash Scripts/install_mlx_lm.sh

# Build CLI from terminal
swift build --target ollmlx

# Build all targets (SPM)
swift build

# Regenerate Xcode project after changing project.yml
xcodegen generate

# Release build with code signing
xcodebuild -scheme OllmlxApp -configuration Release \
  -destination "platform=macOS" \
  DEVELOPMENT_TEAM=M4RUJ7W6MP \
  build

# Build DMG for distribution
bash Scripts/build_dmg.sh
```

---

## Implementation phases

Work through phases in order. Do not start the next phase until all completion criteria for the current phase are checked off. Each phase is documented in full in `ollmlx-implementation-plan.md`.

1. **Core foundation** — OllmlxCore compiles; ServerManager spawns/kills mlx_lm.server
2. **Control API daemon** — DaemonServer serves all five `/control/*` endpoints on `:11435`
3. **Proxy server** — `:11434` transparently forwards to mlx_lm.server's ephemeral port
4. **CLI** — All nine commands work against a running daemon
5. **Menubar app** — Full menubar UI backed by live ServerManager state
6. **Settings** — All settings persist and take effect immediately
7. **Bootstrap and CLI installation** — First-launch experience; CLI symlink to `/usr/local/bin`
8. **Polish and distribution** — Codesigning, notarisation, Sparkle, DMG

---

## Distribution

| Field | Value |
|---|---|
| Bundle ID | `com.darrylmorley.ollmlx` |
| Team ID | `M4RUJ7W6MP` |
| Signing identity | `Developer ID Application: Darryl Morley (M4RUJ7W6MP)` |
| Sparkle feed | `https://github.com/darrylmorley/ollmlx/releases/latest/download/appcast.xml` |
| Sparkle EdDSA key | `m/WL9PKIyMMY1Nx5dL9RzE3GqA+3FlR6OiWTC1IyCfA=` |

Code signing is configured per-build-configuration in `project.yml`:
- **Debug**: `CODE_SIGN_STYLE: Automatic`, identity `-` (ad-hoc)
- **Release**: `CODE_SIGN_STYLE: Manual`, identity `Developer ID Application`

After editing `project.yml`, always run `xcodegen generate` to rebuild `ollmlx.xcodeproj`.

---

## Critical rules

### Threading

- `ServerManager` is `@MainActor` — all state mutations happen on the main actor
- `OllmlxConfig` uses raw `UserDefaults` (not `@AppStorage`) so it is safe to read from CLI contexts and background tasks
- CLI commands run in async contexts — use `Task { @MainActor in }` when touching `ServerManager` from a non-main-actor context
- Never access `ServerManager.shared` from background threads without `await`

### Target boundaries

- **`OllmlxCore` must have ZERO AppKit or SwiftUI imports** — it must compile as a CLI dependency
- **The CLI target must have ZERO AppKit imports**
- Shared types (models, errors, config) belong in `OllmlxCore`
- `@AppStorage` is **banned** in `OllmlxCore` — use `OllmlxConfig` instead

### App lifecycle

- **Single instance**: `applicationDidFinishLaunching` must check `NSRunningApplication.runningApplications(withBundleIdentifier:)` — if `count > 1`, activate the existing instance and terminate self
- **Daemon auto-start**: `DaemonServer` and `ProxyServer` are started automatically in `applicationDidFinishLaunching` via `Task.detached`
- **Bootstrap detection**: Check both `OllmlxConfig.pythonPath` AND the default venv path `~/.ollmlx/venv/bin/python` — if either exists on disk, skip bootstrap and set config
- **Settings window**: Always open as a standalone `NSWindow` via `AppDelegate` — **never** as a `.sheet()` on the NSPopover (sheets on popovers deadlock the entire app, and crash on macOS 26 Tahoe)
- **Pull Model window**: Same as Settings — always open as a standalone `NSWindow` via `AppDelegate` notification (`.openPullModel`), never as a `.sheet()` on the popover
- **Model selector**: The dropdown only updates a `@State` selection — it must **never** call `ServerManager.start()`. Starting/switching happens only when the user clicks the Start button. `MenuBarView` uses `.onChange(of: serverManager.state)` to sync the selected model when external clients trigger a model switch

### Security

- API key **always** stored in Keychain via `Keychain.swift` — **never** in UserDefaults
- `allowExternalConnections = true` requires a non-nil API key — enforce in `DaemonServer` before binding to `0.0.0.0`
- The proxy on `:11434` must validate the API key **before** forwarding, not after

### Process management


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [darrylmorley/ollmlx](https://github.com/darrylmorley/ollmlx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
