---
trigger: always_on
description: This repo is a monorepo for controlling Codex from a phone.
---

# AGENTS

## Purpose

This repo is a monorepo for controlling Codex from a phone.

- Primary product path:
  - `apps/mobile`: Expo React Native client
  - `services/rust-bridge`: current backend bridge (`codex app-server` adapter + terminal/git/attachments/voice helpers)
  - `bin/clawdex.js` + `scripts/*`: operator CLI and setup/runtime automation
- Legacy/reference path:
  - `services/mac-bridge`: older TypeScript bridge with useful tests/reference code, but not the primary runtime

The bridge is intended for trusted/private networks only. Do not treat this repo as internet-safe by default.

## Read First

Use the existing docs as the source of truth instead of duplicating them in code comments or PR notes.

- Quick start and command map: `README.md`
- Setup, secure env flow, verification, smoke tests, API summary: `docs/setup-and-operations.md`
- Troubleshooting and recovery commands: `docs/troubleshooting.md`
- Realtime/live-sync constraints: `docs/realtime-streaming-limitations.md`
- Voice transcription architecture: `docs/voice-transcription.md`
- EAS and native build/release notes: `docs/eas-builds.md`
- Open-source and notice obligations: `docs/open-source-license-requirements.md`
- App review template: `docs/app-review-notes.md`
- App-server/CLI parity tracker: `docs/codex-app-server-cli-gap-tracker.md`

`docs/plans/*` are historical design/implementation plans, not current operating policy.

## Repo Map

### Active code

- `apps/mobile`
  - `App.tsx`: app shell, drawer navigation, persisted settings
  - `src/api/*`: bridge client, websocket transport, typed contracts
  - `src/screens/*`: main UI surfaces
  - `src/components/*`: chat UI pieces
  - `ios/*`: active Expo native iOS project
  - `plugins/withAndroidCleartextTraffic.js`: Android manifest patch for local/insecure bridge access
- `services/rust-bridge`
  - `src/main.rs`: main Axum server, JSON-RPC router, app-server bridge, replay/live-sync logic
  - `src/services/git.rs`: git helpers
  - `src/services/terminal.rs`: terminal execution helpers
- `scripts/*`
  - secure setup/start helpers, Expo bootstrap, service stop/cleanup, version sync
- `.github/workflows/*`
  - CI, npm release, and Pages publishing
- `site/*`
  - static support/privacy/terms site

### Legacy or easy-to-confuse paths

- `services/mac-bridge/*`: legacy TypeScript bridge; useful for reference and tests, not the default runtime
- `ios/*` at repo root: older native iOS tree (`codexmobilecontrol`), not the active Expo app path
- `apps/mobile/ios/*`: this is the active iOS native project for the shipped mobile app
- `apps/telegram-miniapp/*`: currently not a primary maintained source tree; it mostly contains built output/environment leftovers

### Generated/vendor paths to avoid editing by hand

- `node_modules/*`
- `.expo/*`
- `apps/mobile/ios/Pods/*`
- `ios/Pods/*`
- `ios/build/*`
- `apps/telegram-miniapp/dist/*`

## Current Architecture

### Mobile app

- The mobile app is a custom shell, not React Navigation based.
- `apps/mobile/App.tsx` creates exactly one `HostBridgeWsClient` and one `HostBridgeApiClient`, persists app settings, owns the custom drawer, and switches screens via local state.
- The primary screens are:
  - `src/screens/MainScreen.tsx`
  - `src/screens/GitScreen.tsx`
  - `src/screens/SettingsScreen.tsx`
  - `src/screens/OnboardingScreen.tsx`
  - `src/screens/PrivacyScreen.tsx`
  - `src/screens/TermsScreen.tsx`
- `src/screens/TerminalScreen.tsx` exists but is not currently routed from `App.tsx`.
- `src/screens/MainScreen.tsx` is very large and is the main product surface. Treat edits there surgically.

### Bridge/runtime

- The supported backend is `services/rust-bridge`.
- The bridge exposes:
  - `GET /health`
  - `GET /rpc` for WebSocket JSON-RPC
  - `GET /local-image` for mobile image rendering of local/absolute paths
- The Rust bridge spawns `codex app-server --listen stdio://` and forwards an allowlist of `thread/*`, `turn/*`, `review/start`, `model/list`, `skills/list`, `app/list`, and related methods.
- Bridge-native RPC methods include attachments upload, voice transcription, terminal exec, git operations, approvals, user-input resolution, and event replay.

### Realtime model

- Mobile realtime is hybrid:
  - live WS notifications when the bridge owns the stream
  - replay buffer recovery via `bridge/events/replay`
  - snapshot/poll convergence for persisted history
  - rollout/session tailing in Rust for best-effort CLI-origin live sync
- If work touches missing live updates, read `docs/realtime-streaming-limitations.md` before changing the bridge or mobile sync loop.

## Primary Workflows

### Preferred operator flow

- Published CLI:
  - `clawdex init`
  - `clawdex stop`
- Monorepo equivalents:
  - `npm run setup:wizard`
  - `npm run stop:services`

### Root scripts

From repo root:

- `npm run mobile`
- `npm run ios`
- `npm run android`
- `npm run bridge`
- `npm run bridge:ts`
- `npm run secure:setup`
- `npm run secure:bridge`
- `npm run secure:bridge:dev`
- `npm run teardown`
- `npm run lint`
- `npm run typecheck`
- `npm run build`
- `npm run test`
- `npm run version:sync`

### Important operational details


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mohit-Patil/clawdex-mobile](https://github.com/Mohit-Patil/clawdex-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
