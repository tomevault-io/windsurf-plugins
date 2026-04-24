---
trigger: always_on
description: Music Quiz v4.0.0 — multiplayer music quiz party game powered by Apple Music.
---

# Music Quiz — Project Instructions

## Current Status (April 2026)

Music Quiz v4.0.0 — multiplayer music quiz party game powered by Apple Music.
Core quiz + DJ Mode + Party Sessions + MusicKit JS + AI Trivia working end-to-end.

## Monorepo Layout (v4)

pnpm + Turborepo workspace. Run `pnpm install` then `pnpm build` from the root.

```
packages/
  shared/        @music-quiz/shared      — pure types (was src/quiz/types.ts)
  quiz-engine/   @music-quiz/quiz-engine — game engine, Express routes, WS handlers,
                                            Apple Music client, OAuth, MCP token,
                                            home-ws, browser-ws, playback providers,
                                            and the vanilla quiz UI under src/public/
  mcp-server/    @music-quiz/mcp-server  — MCP entry point + Express/Next bootstrap.
                                            Contains src/index.ts and server.js.
  web/           @music-quiz/web         — Next.js 16 frontend (App Router)
home/            standalone Mac agent, NOT in the workspace (own package.json)
apps/tvos/       NATIVE SwiftUI display client (Apple TV) — TestFlight live, NOT in workspace
```

**Dependency graph (one-way, no cycles):**
- `shared` → no deps
- `quiz-engine` → `shared`
- `mcp-server` → `quiz-engine` + `shared`
- `web` → `shared`

**Deviation from the original migration doc:** the doc placed `apple-music.ts`,
`oauth.ts`, `token.ts`, `token-store.ts`, `browser-ws.ts`, `home-ws.ts`,
`quiz.ts` and `quiz-manager.ts` inside `mcp-server`. Doing so would have
created a circular dependency (quiz-engine needs all of those, and mcp-server
imports from quiz-engine). They now all live in `quiz-engine` and are
re-exported via `packages/quiz-engine/src/index.ts` so `mcp-server` consumes
everything through the single workspace name `@music-quiz/quiz-engine`.

**Key build commands:**
- `pnpm install`                       — install all workspace deps
- `pnpm build`                         — turbo build (shared → quiz-engine → mcp-server + web)
- `pnpm --filter @music-quiz/mcp-server dev`  — start backend (tsc + node server.js)
- `pnpm --filter @music-quiz/web dev`         — Next.js dev server
- `pnpm start`                         — production node packages/mcp-server/server.js

`home/` is built independently with its own `tsconfig.json` + `package.json`
(plain `npm install && npx tsc` inside `home/`).

## tvOS App (apps/tvos/) — Status (April 2026)

**Live on TestFlight as build 0.1.0 (2)** under WebHouse Apple Developer team
`7NAG4UJCT9`. Bundle ID `dk.webhouse.music-quiz.tvos`. Confirmed running on
real Apple TV hardware. Display name in App Store Connect: **Quiz Mash Music**.

### Architecture
- 100% **native SwiftUI** (NOT a WKWebView shell — WKWebView doesn't exist on
  tvOS, that was the original plan and got revised in Phase 2-ny).
- **Display client only** — read-only spectator. Connects to `/quiz-ws` as
  `register_display` role. Never sends host commands. All quiz control still
  happens via web admin / mobile PWA.
- **Native MusicKit playback** via `MusicKit.framework` (when entitlement is
  re-enabled — see "Outstanding issues" below). Acts as the consumer of the
  existing `MusicKitWebProvider` outbound channel — zero server-side changes
  needed for tvOS-specific routing.
- See `docs/TVOS-WS-PROTOCOL.md` for the canonical wire contract (owned by
  the web/server side).

### Implemented features
- All quiz states render: Idle, Lobby (QR + players + join code),
  Countdown, Question (artwork + 4 colored options), Reveal (answer card +
  fun fact), Scoreboard (gold/silver/bronze ranks), Final Podium (with
  confetti + gold-pulse), DJ Mode (queue + per-player picks), Now Playing
  (vinyl sphere with pulsing red glow halo).
- Blind Round + Steal Round (april 2026 game modes) — banner badges,
  hidden options when blind, 5s steal countdown ring with pulsing red
  banner, steal-winner highlight in reveal.
- WebSocket auto-reconnect with exponential backoff.
- Demo mode (`SIMCTL_CHILD_MUSIC_QUIZ_DEMO=1`) cycles through every state
  in ~50s for visual smoke tests without a server.
- App icon + Top Shelf (regular + wide) generated from
  `logo/music-quiz-logo.svg` via `pnpm tvos:icon`.
- Fastlane pipeline: `pnpm tvos:build` (unsigned simulator smoke build),
  `pnpm tvos:beta` (full TestFlight upload via cert+sigh+gym).

### Outstanding issues
1. ~~**MusicKit entitlement is currently DISABLED**~~ — **RESOLVED april 2026.**
   `com.apple.developer.musickit` is **NOT required for tvOS** — that
   entitlement is iOS / macOS only. The previous session chased a phantom
   requirement based on iOS docs; the "Apple bug" where sigh's profiles
   strip the entitlement is actually correct Apple behaviour. Verified by
   Xcode 26's capability picker which lists no MusicKit option when the
   target platform is tvOS (only Sign In with Apple, App Groups, etc.).

   **What tvOS apps actually need for MusicKit:**
   - `MusicKit.framework` linked in the build target
   - `NSAppleMusicUsageDescription` in Info.plist (already present)
   - Runtime authorization via `MusicAuthorization.request()` in Swift
   - **No entitlement key** in `.entitlements` file
   - The `MusicQuiz.entitlements` file is intentionally empty

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cbroberg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
