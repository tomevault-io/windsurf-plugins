---
trigger: always_on
description: Cross-platform desktop app for **real-time translation of system audio** (meetings, videos, calls). **Named "Earshot" on 2026-07-24** (previously "Real-Time Translator") — the display string; the npm slug `claude-real-time-translator` and the `appId` `com.amir.realtimetranslator` deliberately keep the old name, since changing the `appId` would orphan existing installs. **The GitHub repo WAS renamed to `Earshot` on 2026-07-27** (github.com/amirmokarram/Earshot; GitHub redirects the old clone URL,
---

# Earshot — Project Context

Cross-platform desktop app for **real-time translation of system audio** (meetings, videos, calls). **Named "Earshot" on 2026-07-24** (previously "Real-Time Translator") — the display string; the npm slug `claude-real-time-translator` and the `appId` `com.amir.realtimetranslator` deliberately keep the old name, since changing the `appId` would orphan existing installs. **The GitHub repo WAS renamed to `Earshot` on 2026-07-27** (github.com/amirmokarram/Earshot; GitHub redirects the old clone URL, `origin` updated), and `electron-builder.json`'s publish block was updated to `Earshot` so CI releases still target the right repo. The user (Amir) listens to audio in one language and reads a live translation in another. The **language pair is fully user-configurable** (Settings → Languages) — any supported source→target pair from a curated catalog.

> Full detail lives in [`docs/memory/`](docs/memory/). This file is the auto-loaded summary — it travels with the repo, so project context survives a machine/OS change. The canonical copies I auto-read also live in `~/.claude/projects/D--Claude-RealTimeTranslator/memory/`; keep the two in sync (treat `~/.claude` as source of truth I edit, then copy here before committing).

## Stack
- **Angular 21** (standalone components, signals, `@if`/`@for`) — renderer
- **Electron 43** — desktop shell
- 2 switchable streaming STT providers: **DeepGram** (cloud, WebSocket) and **Whisper** (local, WhisperLive WebSocket) — renderer-side `ISttStream` strategy
- **Providers are data, not code** (Phase 14): one catalog serves BOTH translation and assist. `electron/config/providers.json` (+ an optional `<userData>/providers.json` overlay, **never containing credentials**) declares each provider over one of 8 built-in wire-protocol **adapters** in `electron/providers/adapters/`. Ships Claude, OpenAI, Ollama, OpenAI-compatible, Google, DeepL, Microsoft, LibreTranslate — see [`docs/memory/provider-catalog.md`](docs/memory/provider-catalog.md)
- Configurable source/target languages from a curated catalog — **single source of truth `src/app/core/models/languages.json`** (renderer imports it; the build copies it to `dist-electron/config/` so the main process reads it at runtime via `electron/languages.ts`); per-cell text direction/font driven by each language's `rtl` flag (Vazirmatn for RTL)
- Dark theme; Vazirmatn font for Persian/RTL text

## Architecture (key rules)
- **Translation & assist API keys + calls stay in the Electron MAIN process** — never move them to Angular services. Keys live in `settings.providerConfigs[providerId]` (userData), **one bag per provider shared by both features**; the catalog JSON is refused outright if it contains a key-shaped field anywhere.
- **A provider is a catalog entry naming an adapter.** Adding anything with an OpenAI-shaped API (Groq, OpenRouter, Together, DeepSeek, Azure OpenAI, LM Studio, vLLM) is a JSON edit over the `openai-chat` adapter — no code. Only a genuinely new wire protocol needs an adapter class. Never put provider knowledge in the renderer; if a panel needs to know something about a provider, it belongs in the descriptor.
- **STT streaming is the exception: it runs in the RENDERER** (browser `WebSocket`, like DeepGram's subprotocol-token auth). Both providers live behind a renderer-side `ISttStream` strategy (`src/app/core/services/stt/`); `TranscriptionService` owns sentence segmentation and picks the strategy from `stt.provider`. Whisper uses a local WhisperLive WS — see [`docs/memory/whisper-stt-provider.md`](docs/memory/whisper-stt-provider.md).
- Renderer ↔ main via secure IPC bridge (`contextBridge` + `electron/preload.ts`, typed `ElectronAPI`).
- Router uses **HashLocationStrategy** (required for `file://` prod load; also how the overlay targets `#/overlay`).
- Translation events (`translation:source`/`:chunk`/`:complete`) are **broadcast to all windows** so the overlay mirrors the main window for free.
- Settings persist to `userData/settings.json`. The `AppSettings` schema is defined once in `shared/app-settings.d.ts` (shared across both TS contexts); defaults live in `electron/config/default-settings.json`.
- TS isolation: `tsconfig.app.json` (renderer, browser types) vs `tsconfig.electron.json` (main, CommonJS/Node).

## Build & run (Windows / PowerShell)
- `npm run electron:dev` — dev (ng serve :4200 + Electron hot-reload)
- `npm run electron:compile` — compile Electron TS only → `dist-electron/`
- `npm run electron:dist:win` — package Windows installer
- After any **main-process** change: recompile Electron + restart.

## Critical gotchas (see docs/memory/gotchas-and-lessons.md)
- **Web Speech API does NOT work in Electron** (no Google keys → always `error: network`). Use DeepGram.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amirmokarram/Earshot](https://github.com/amirmokarram/Earshot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
