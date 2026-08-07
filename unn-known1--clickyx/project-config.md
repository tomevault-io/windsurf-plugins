---
trigger: always_on
description: ClickyX is a cross-platform AI desktop companion — voice, screen context, cursor overlay, background agents, computer use, and a local HTTP bridge. Built with **Tauri v2 (Rust) + React 19 + TypeScript**. Runs on Windows, Linux, and macOS. Zero cloud dependency, zero telemetry.
---

# ClickyX Agent Instructions

## Project Overview

ClickyX is a cross-platform AI desktop companion — voice, screen context, cursor overlay, background agents, computer use, and a local HTTP bridge. Built with **Tauri v2 (Rust) + React 19 + TypeScript**. Runs on Windows, Linux, and macOS. Zero cloud dependency, zero telemetry.

Full specification: `docs/PROJECT_SPEC.md` — the single source of truth for features, architecture, and implementation details.

---

## Key Rules

1. **Cross-platform first** — never write platform-specific code without equivalent implementations for all 3 platforms
2. **No macOS lock-in** — no Foundation, SwiftUI, AppKit, or Apple-only framework
3. **Local-first** — API keys are user-configured; no cloud sync, no hosted OAuth, no telemetry
4. **Bridge compatibility** — `localhost:32123` HTTP API must stay compatible with OpenClicky's spec
5. **Use AppContext** — all toasts and navigation go through `src/context/AppContext.tsx`, never `window.__`
6. **Use typed bindings** — all `invoke()` calls must reference `src/bindings.ts`
7. **Use react-query** — all server data fetching uses `useQuery`/`useMutation` — no raw `useState+useEffect+invoke` for data
8. **Tests for new hooks** — any new hook under `src/hooks/` must have a `.test.ts` sibling

---

## Technology Stack

| Layer | Technology |
|-------|-----------|
| App shell | Tauri v2 (Rust + WebView) |
| Audio | `cpal` crate |
| Screen capture | `xcap` crate |
| Input simulation | `enigo` crate |
| HTTP bridge | `actix-web` on `127.0.0.1:32123` |
| Frontend | React 19 + TypeScript + Vite |
| Global state | Zustand (`src/store/appStore.ts`) |
| Data fetching | `@tanstack/react-query` |
| i18n | `i18next` + `react-i18next` (EN/ES/FR/JA) |
| Overlay | Per-screen transparent `WebviewWindow` (`src/overlay/`) |
| AI providers | HTTP/WebSocket — Anthropic, OpenAI, Deepgram, ElevenLabs, etc. |
| Agent runtime | Codex (Node.js, already cross-platform) |
| System TTS | `tts` crate — SAPI (Windows), AVFoundation (macOS), Speech Dispatcher (Linux) |

---

## Verification Commands

```sh
cargo check                   # Rust compile check
cargo test --all-features     # Rust unit tests
npm run build                 # tsc + vite (frontend)
npm test                      # Vitest unit tests
npm run test:e2e              # Playwright E2E
npm run test:visual           # Playwright visual regression
```

---

## Key File Map

### Rust Backend (`src-tauri/src/`)

| File / Dir | Purpose |
|-----------|---------|
| `bridge.rs` | HTTP API on `localhost:32123` — 25+ endpoints, MCP real stdio JSON-RPC |
| `bridge_auth.rs` | Constant-time token auth middleware |
| `audio/pipeline.rs` | VAD loop, audio ducking, voice-agent handoff, always-on mode |
| `audio/handoff.rs` | `VoiceAgentHandoff` — phrase detection → `voice-agent-handoff` event |
| `audio/tts.rs` | TTS providers (ElevenLabs, Cartesia, Edge, Deepgram Aura, OpenAI Realtime, System TTS) |
| `audio/stt.rs` | STT providers (Deepgram, Whisper, AssemblyAI) |
| `audio/voices.rs` | 6-provider voice catalog (includes System TTS) |
| `ai/guidance.rs` | Annotation tag parser — POINT, RECT, SCRIBBLE, OFFER, HIGHLIGHT, SHAPE |
| `ai/app_contexts.rs` | Per-app CUA context injection (VS Code, Figma, Terminal, Blender, etc.) |
| `ai/catalog.rs` | Dynamic model catalog |
| `agent/session.rs` | Agent session lifecycle (create/run/stop/archive) |
| `agent/skills.rs` | Skills loader |
| `agent/codex.rs` | Codex sidecar process management |
| `agent/google.rs` | Google Workspace stub — returns `available: false`; OAuth2 not yet wired (gogcli dependency removed) |
| `screen/capture.rs` | xcap-based screenshot capture |
| `screen/auto_capture.rs` | Diff-based continuous capture engine |
| `screen/coordinate.rs` | Y-flip + coordinate normalization |
| `overlay/window_manager.rs` | Per-screen overlay windows + hotplug watcher |
| `overlay/screen_router.rs` | `CoordinateNormalizer`, `ScreenManager` |
| `overlay/lifecycle.rs` | Annotation lifecycle (armed → completed → missed) |
| `overlay/manager.rs` | Annotation manager + sweep task |
| `cua.rs` | `InputSimulator` — click, scroll, type, background mode |
| `permissions.rs` | Real OS checks — TCC sqlite3 (macOS), registry (Windows), pactl (Linux) |
| `automation/mod.rs` | Cron + interval scheduler with JSON persistence |
| `gen3d.rs` | Tripo3D API |
| `updater.rs` | Platform-aware updater with streaming progress events |
| `config.rs` | Config load/save/export/import/reset |
| `commands.rs` | All Tauri command handlers |
| `lib.rs` | App setup, plugin registration, deep-link handler |
| `tray.rs` | System tray setup |
| `type_mode.rs` | Double-tap Ctrl type mode |

### Frontend (`src/`)

| File | Purpose |
|------|---------|
| `App.tsx` | Shell: lazy tabs, splash screen, titlebar drag-region, panel drop zone, deep-link handler, `aria-current` on tabs |
| `AgentHUDApp.tsx` | Floating HUD window entry point |
| `main.tsx` | React 19 root — `QueryClientProvider` + i18n bootstrap |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unn-Known1/clickyX](https://github.com/unn-Known1/clickyX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
