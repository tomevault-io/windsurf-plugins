---
trigger: always_on
description: AivoClaw is a cross-platform desktop app that wraps the [openclaw](https://github.com/anthropics/claude-code) gateway into a standalone installable package. It ships a bundled Node.js 22 runtime and the openclaw npm package, so users need zero dev tooling — just install and run.
---

# AivoClaw — Electron Shell for openclaw

## What This Project Is

AivoClaw is a cross-platform desktop app that wraps the [openclaw](https://github.com/anthropics/claude-code) gateway into a standalone installable package. It ships a bundled Node.js 22 runtime and the openclaw npm package, so users need zero dev tooling — just install and run.

**Three-process architecture:**

```
Electron Main Process
  ├── Gateway child process  (Node.js 22 → openclaw entry.js, port configurable, default 18789)
  └── BrowserWindow          (loads Lit Chat UI via file://, connects to gateway via WebSocket)
```

The main process spawns a gateway subprocess, waits for its health check, then opens a BrowserWindow that loads a local Lit-based Chat UI (via `file://`). The Chat UI connects to the gateway over WebSocket for chat and HTTP for API calls. A system tray icon keeps the app alive when all windows are closed.

## Tech Stack

| Layer | Choice |
|---|---|
| Shell | Electron 40.2.1 |
| Language | TypeScript → CommonJS (no ESM) |
| Chat UI | Lit 3 + Vite (file:// loaded SPA) |
| Packager | electron-builder 26.7.0 |
| Updater | electron-updater (generic provider, CDN at `aivoclaw.cn`) |
| Targets | macOS DMG + ZIP (arm64/x64), Windows NSIS (x64/arm64) |
| Version scheme | Calendar-based: `2026.2.26` (auto-fetched from openclaw npm at build time) |

## Repository Layout

```
aivoclaw/
├── src/                    # 25 TypeScript modules (6571 LOC) + 6 test files
│   ├── main.ts             # App entry, lifecycle, IPC, Dock toggle, config recovery
│   ├── constants.ts        # Path resolution (dev vs packaged), health check params
│   ├── gateway-process.ts  # Child process state machine + diagnostics
│   ├── gateway-auth.ts     # Auth token read/generate/persist
│   ├── gateway-rpc.ts      # WebSocket RPC client for main↔gateway communication
│   ├── window.ts           # BrowserWindow lifecycle, token injection, retry
│   ├── window-close-policy.ts  # Close behavior: hide vs destroy
│   ├── tray.ts             # System tray icon + i18n context menu
│   ├── preload.ts          # contextBridge IPC whitelist (42 methods + 4 listeners)
│   ├── provider-config.ts  # Provider presets, verification, config R/W
│   ├── setup-manager.ts    # Setup wizard window lifecycle
│   ├── setup-ipc.ts        # Setup validation + config write + CLI install
│   ├── setup-completion.ts # Setup wizard completion detection
│   ├── aivoclaw-config.ts   # AivoClaw ownership config (deviceId, setupCompletedAt, migration)
│   ├── settings-ipc.ts     # Settings CRUD, backup/restore, Kimi, CLI, advanced
│   ├── config-backup.ts    # Rolling backups + last-known-good snapshot + restore
│   ├── share-copy.ts       # Remote share copy content (CDN fetch + local fallback)
│   ├── kimi-config.ts      # Kimi robot plugin + Kimi Search configuration
│   ├── cli-integration.ts  # CLI wrapper generation, PATH injection (POSIX + Windows)
│   ├── launch-at-login.ts  # macOS/Windows launch at login toggle
│   ├── feishu-pairing-monitor.ts  # Feishu pairing request polling + state
│   ├── update-banner-state.ts     # Update banner pure state machine
│   ├── analytics.ts        # Telemetry (PostHog-style, retry + fallback URL)
│   ├── analytics-events.ts # Event classification + property sanitization
│   ├── auto-updater.ts     # electron-updater wrapper + progress callback
│   └── logger.ts           # Dual-write logger (file + console)
├── chat-ui/                # Lit-based Chat UI SPA (file:// loaded)
│   └── ui/                 # Vite project: Lit 3 components, sidebar, settings view
├── setup/                  # Setup wizard frontend (vanilla HTML/CSS/JS)
│   ├── index.html          # 3-step wizard with data-i18n attributes
│   ├── setup.css           # Dark/light theme via prefers-color-scheme
│   └── setup.js            # i18n dict (en/zh) + form logic
├── settings/               # Settings page frontend (vanilla HTML/CSS/JS)
│   ├── index.html          # Provider, Search, Channels, KimiClaw, Appearance, Advanced, Backup tabs
│   ├── settings.css        # Dark/light theme via prefers-color-scheme
│   └── settings.js         # Provider CRUD, Feishu, Kimi, Kimi Search, CLI, backup/restore
├── scripts/
│   ├── package-resources.js    # Downloads Node.js 22 + installs openclaw from npm
│   ├── afterPack.js            # electron-builder hook: injects resources post-strip
│   ├── run-mac-builder.js      # macOS build wrapper (sign + notarize)
│   ├── run-with-env.js         # .env loader for child processes
│   ├── merge-release-yml.js    # Merges per-arch latest.yml for auto-updater
│   ├── dist-all-parallel.sh    # Parallel cross-platform build
│   └── clean.sh
├── assets/                 # Icons: .icns, .ico, .png, tray templates
├── docs/                   # Plans and documentation
├── .github/workflows/      # CI: build-release.yml + publish-release.yml + publish-share-copy.yml
├── electron-builder.yml    # Build config (DMG + ZIP for mac, NSIS for win)
├── tsconfig.json           # target ES2022, module CommonJS

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yuanyuekejiJN/AivoClaw](https://github.com/yuanyuekejiJN/AivoClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
