---
trigger: always_on
description: JSTorrent is a multi-platform BitTorrent client with a **Chrome MV3 extension** frontend and **Rust native components** for privileged I/O operations. The monorepo is organized into distinct components that communicate via native messaging and HTTP/WebSocket protocols.
---

# JSTorrent Monorepo AI Coding Guide

## Architecture Overview

JSTorrent is a multi-platform BitTorrent client with a **Chrome MV3 extension** frontend and **Rust native components** for privileged I/O operations. The monorepo is organized into distinct components that communicate via native messaging and HTTP/WebSocket protocols.

### Component Boundaries

```
extension/          Chrome MV3 extension (TypeScript + React + Vite)
├─ Service worker orchestrates torrent engine + native communication
├─ UI components use React with no HMR (MV3 CSP constraints)
└─ Connects to desktop via chrome.runtime.connectNative

desktop/            Rust workspace with 4 packages
├─ common/              Shared library (jstorrent_common)
├─ host/                Chrome native messaging coordinator (jstorrent-host)
├─ io-daemon/           HTTP/WebSocket server for file/socket I/O
└─ link-handler/        OS magnet://.torrent protocol handler

packages/engine/    Core BitTorrent engine (TypeScript, no dynamic imports)
packages/shared-ts/ Shared TypeScript types and utilities
packages/proto/     Protocol definitions (future)
```

**Critical architectural invariants:**
- Extension contains torrent logic; native components only do I/O
- Native-host spawns io-daemon as a child process (shares lifecycle)
- IO daemon never writes config; only native-host manages `rpc-info.json`
- Link handler cannot start native-host directly (only via browser LAUNCH_URL)

## Build & Development Commands

**From monorepo root (pnpm workspace):**
```bash
pnpm install                    # Install all workspace dependencies
pnpm checkall                   # Parallel: lint, format, typecheck, test
pnpm build                      # Build all packages
pnpm --filter extension build   # Build specific package
pnpm --filter extension test:e2e  # Run Playwright tests
```

**Extension-specific (in extension/):**
```bash
pnpm dev            # Watch mode (vite build --watch, no HMR)
pnpm check_fast     # Fast checks: lint, format, typecheck, unit tests
pnpm test:e2e       # Playwright integration tests (requires native-host)
```

**System Bridge (in desktop/):**
```bash
cargo build --release --workspace           # Build all binaries
cargo test --workspace                      # Run Rust tests
python3 verify_all.py                       # Integration tests (simulates extension)
./scripts/install-local-linux.sh            # Install for local testing
./scripts/install-local-macos.sh            # macOS equivalent (requires sudo)
```

**Load extension in Chrome:** Build first (`pnpm build`), then load `extension/dist/` as unpacked extension.

## Testing Strategy

1. **Unit tests**: Vitest with happy-dom + React Testing Library
   - Mock chrome.* APIs via `test/mocks/mock-chrome.ts`
   - Run with `pnpm test` (in extension/)

2. **Integration tests**: Playwright in "new headless" mode
   - Load extension via `--load-extension=dist --disable-extensions-except=dist`
   - Requires native-host installed locally
   - Run with `pnpm test:e2e`

3. **Native verification scripts**: Python scripts that simulate extension behavior
   - `verify_all.py` runs all `verify_*.py` scripts
   - Tests native components in isolation without browser

4. **DO NOT** run `jupyter notebook` or browser-based commands for testing

## Project-Specific Conventions

### TypeScript
- **Strict mode enabled**: No implicit any, unused locals/params flagged
- **No dynamic imports in `packages/engine/`** (ESLint enforces this)
- Use `tsx` for running REPL: `pnpm --filter @jstorrent/engine repl`

### Linting & Formatting
- ESLint config at root (`eslint.config.js`) uses flat config format
- Prettier must come **last** in ESLint config to override formatting rules
- WebExtensions globals pre-configured (`chrome.*` available without imports)
- Unused variables with `_` prefix are allowed: `const _unused = foo()`

### Native Messaging & RPC
- Extension talks to native-host via `chrome.runtime.connectNative('com.jstorrent.native')`
- Native-host returns io-daemon port + auth token via structured JSON messages
- IO daemon uses **download root tokens** (opaque SHA1-based) instead of raw paths
- All daemon requests include `root_token` + `relative_path` for security

### File Paths & Discovery
- **Config dir** (Linux): `~/.config/jstorrent-native/`
- **macOS**: `~/Library/Application Support/jstorrent-native/`
- **Windows**: `%LOCALAPPDATA%\jstorrent-native\`
- **RPC info file**: `rpc-info.json` (written only by native-host, read by daemon/link-handler)

### Vite Build
- Multi-entry build for MV3 pages: `src/sw.ts`, `src/ui/app.html`, etc.
- **No HMR or dev server** (MV3 CSP incompatible)
- Sourcemaps enabled, non-minified output for debugging
- Build outputs to `extension/dist/`

## CI/CD Pipeline

- **Extension CI** (`.github/workflows/extension-ci.yml`):
  - Triggers on `extension/**` or `packages/**` changes
  - Builds extension, installs native-host, runs Playwright tests

- **System Bridge CI** (`.github/workflows/system-bridge-ci.yml`, builds `desktop/`):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kzahel/JSTorrent](https://github.com/kzahel/JSTorrent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
