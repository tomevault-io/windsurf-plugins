---
trigger: always_on
description: OneClaw is a cross-platform desktop app that wraps the [openclaw](https://github.com/openclaw/openclaw) gateway into a standalone installable package. It ships a bundled Node.js 22 runtime and the openclaw npm package, so users need zero dev tooling — just install and run.
---

# OneClaw — Electron Shell for openclaw

## What This Project Is

OneClaw is a cross-platform desktop app that wraps the [openclaw](https://github.com/openclaw/openclaw) gateway into a standalone installable package. It ships a bundled Node.js 22 runtime and the openclaw npm package, so users need zero dev tooling — just install and run.

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
| Updater | electron-updater (generic provider, CDN at `oneclaw.cn`) |
| Targets | macOS DMG + ZIP (arm64/x64), Windows NSIS (x64/arm64) |
| Version scheme | Calendar-based: `YYYY.MMDD.N` (e.g. `2026.318.0`), auto-derived from git tag |

## Repository Layout

```
oneclaw/
├── src/                    # 35 TypeScript modules (10270 LOC) + 13 test files
│   ├── main.ts             # App entry, lifecycle, IPC, Dock toggle, config recovery
│   ├── constants.ts        # Path resolution (dev vs packaged vs ASAR), health check params
│   ├── gateway-process.ts  # Child process state machine + diagnostics
│   ├── gateway-auth.ts     # Auth token read/generate/persist
│   ├── gateway-rpc.ts      # WebSocket RPC client for main↔gateway communication
│   ├── window.ts           # BrowserWindow lifecycle, token injection, retry
│   ├── window-close-policy.ts  # Close behavior: hide vs destroy
│   ├── tray.ts             # System tray icon + i18n context menu
│   ├── preload.ts          # contextBridge IPC whitelist (~75 methods + 5 listeners)
│   ├── provider-config.ts  # Provider presets, verification, config R/W
│   ├── setup-manager.ts    # Setup wizard window lifecycle
│   ├── setup-ipc.ts        # Setup validation + config write + CLI install
│   ├── setup-completion.ts # Setup wizard completion detection
│   ├── install-detector.ts # Setup Step 0: installation conflict detection
│   ├── oneclaw-config.ts   # OneClaw ownership config (deviceId, setupCompletedAt, migration)
│   ├── settings-ipc.ts     # Settings CRUD, backup/restore, Kimi, CLI, advanced
│   ├── config-backup.ts    # Rolling backups + last-known-good snapshot + restore
│   ├── share-copy.ts       # Remote share copy content (CDN fetch + local fallback)
│   ├── kimi-config.ts      # Kimi robot plugin + Kimi Search configuration
│   ├── kimi-oauth.ts       # Kimi OAuth device code login + token refresh
│   ├── skill-store.ts      # Skill marketplace (clawhub CLI integration)
│   ├── build-config.ts     # Build-time injected config reader (PostHog, registry URL)
│   ├── cli-integration.ts  # CLI wrapper generation, PATH injection (POSIX + Windows)
│   ├── launch-at-login.ts  # macOS/Windows launch at login toggle
│   ├── channel-pairing-monitor.ts  # Unified multi-channel pairing polling + state
│   ├── channel-pairing-store.ts    # Per-channel pairing approval persistence
│   ├── feishu-pairing-monitor.ts   # Feishu-specific pairing monitor
│   ├── wecom-config.ts     # WeCom (企业微信) plugin config
│   ├── weixin-config.ts    # WeChat (微信) plugin config
│   ├── dingtalk-config.ts  # DingTalk connector plugin config
│   ├── qqbot-config.ts     # QQ Bot plugin config
│   ├── update-banner-state.ts     # Update banner pure state machine
│   ├── analytics.ts        # Telemetry (PostHog-style, retry + fallback URL)
│   ├── analytics-events.ts # Event classification + property sanitization
│   ├── auto-updater.ts     # electron-updater wrapper + progress callback
│   └── logger.ts           # Dual-write logger (file + console)
├── chat-ui/                # Lit-based Chat UI SPA (file:// loaded, ~35K LOC)
│   └── ui/                 # Vite project: Lit 3 components, sidebar, settings view, model selector
├── setup/                  # Setup wizard frontend (vanilla HTML/CSS/JS)
│   ├── index.html          # Multi-step wizard with data-i18n attributes
│   ├── setup.css           # Dark/light theme via prefers-color-scheme
│   ├── setup.js            # i18n dict (en/zh) + form logic
│   └── lucide-sprite.generated.js  # Icon sprites
├── settings/               # Settings page frontend (vanilla HTML/CSS/JS)
│   ├── index.html          # Provider, Search, Channels, KimiClaw, Appearance, Advanced, Backup tabs
│   ├── settings.css        # Dark/light theme via prefers-color-scheme
│   ├── settings.js         # Provider CRUD, multi-channel, Kimi, CLI, backup/restore
│   ├── lucide-sprite.generated.js  # Icon sprites

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oneclaw/oneclaw](https://github.com/oneclaw/oneclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
