---
trigger: always_on
description: Sei is a Minecraft AI companion. This repository is the **client**: an Electron
---

# Sei — Contributor Guide

Sei is a Minecraft AI companion. This repository is the **client**: an Electron
desktop app ("Sei", productName in electron-builder.yml) for non-technical users that spawns an AI-driven
[mineflayer](https://github.com/PrismarineJS/mineflayer) bot into a **LAN
(offline-mode) Minecraft Java world**. You pick a character, the bot joins your
world, and it talks and plays alongside you.

v1.0 is LAN-worlds only — offline mode, no Mojang/Microsoft auth, no Mojang
UUIDs. Identity is the in-game username plus (for cloud users) a Supabase
account.

> **Scope note.** This repo is the client only. The cloud backend it talks to
> (the LLM proxy, Supabase database, billing webhooks) is a **separate private
> service**. Everything here that mentions "the proxy" or "the server" refers to
> that external component — there is no server code in this tree.

---

## Architecture: three-process Electron

Electron is split into three trust zones plus a forked bot subprocess. The
boundaries are load-bearing — respect them.

```
┌───────────────┐  IPC (contextIsolation)   ┌────────────────┐
│   renderer    │ ───── window.sei ───────▶ │      main      │
│  React 19 +   │ ◀──── (preload bridge) ─── │  Electron host │
│   Zustand     │                            └───────┬────────┘
└───────────────┘                                    │ utilityProcess.fork
   src/renderer            src/preload                │ + MessageChannelMain
                                                      ▼
                                            ┌────────────────┐
                                            │   bot (LLM +   │
                                            │   mineflayer)  │
                                            └────────────────┘
                                                  src/bot
```

| Process | Source | Role |
|---|---|---|
| **main** | `src/main/` (entry `src/main/index.ts`) | Electron host: window, IPC, stores, auth, cloud, updater, bot supervisor. The only process that touches the OS keychain and the network for cloud/auth. |
| **renderer** | `src/renderer/` | React 19 + Zustand UI. `contextIsolation` is ON; it has **no Node access** and reaches main **only** through the `window.sei` bridge. |
| **preload** | `src/preload/index.ts` | Typed `RendererApi` over `ipcRenderer.invoke`, exposed as `window.sei` via `contextBridge`. Compiled to **`.cjs`**. |
| **bot** | `src/bot/` | The companion: LLM brain + mineflayer. Forked by `src/main/botSupervisor.ts` via `utilityProcess.fork`, talks to main over `MessageChannelMain`. |

### Invariants (do not break these)

- **mineflayer is imported only in `src/bot`.** It must run in the
  utilityProcess, never in main or renderer.
- **The renderer never imports from `src/main`.** All renderer→main traffic goes
  through `window.sei` (preload) → IPC channels declared in `src/shared/ipc.ts`.
- **Plaintext secrets cross to the bot only over `MessagePortMain`**, never
  through the renderer. `src/main/apiKeyStore.ts` decrypts the API key in main
  and hands it to the forked bot in the init message.
- **Multiple bots, one per character.** `botSupervisor.ts` owns a
  `Map<characterId, ActiveSession>` — `summon(id)` forks an *additional* bot
  without disturbing the others; `stop(id)` drains one, `stop()`/`shutdown()`
  drain all. Each character is its own `utilityProcess` + brain + memory dir, so
  sessions are fully independent. **Two bots may never share an in-game
  username** (the world kicks the second with `name_taken`), so `summon` refuses
  a colliding effective username before forking (the renderer pre-checks and
  shows a popup; the supervisor is the authoritative backstop). Summon has a
  hard **30s timeout** (`SUMMON_TIMEOUT_MS`); stop has a 10s timeout then
  escalates to kill. The in-game username is `effectiveMcUsername(character)` in
  `src/shared/characterSchema.ts` (`character.username` ?? sanitized name).
- IPC contracts and shared Zod schemas live in `src/shared` and are the single
  source of truth for both sides of the bridge.

---

## Local vs Cloud mode

The bot reaches an LLM through one of two backends, selected by
`ai_backend_kind` in `<userData>/config.json` (**default `'local'`**), read via
`getAiBackendKind()` in `src/main/apiKeyStore.ts`.

| | **local** (BYOK) | **cloud-proxy** |
|---|---|---|
| Auth | User's own Anthropic API key, encrypted at rest via Electron `safeStorage` (OS keychain) | Supabase account; JWT (`access_token`) sent as a Bearer token |
| Endpoint | Anthropic direct | `https://api.sei.gg` (the private proxy) |
| Credits UI | Hidden | Pricing / credits / hard-stop surfaces shown |

**Runtime wiring** lives in `src/bot/brain/anthropicClient.js` →
`buildSdkOptions()`:

- **local:** `{ apiKey: <decrypted key> }`.
- **cloud:** `{ baseURL, authToken, apiKey: null }`. Passing `apiKey: null` is
  deliberate — it suppresses the `x-api-key` header so only the
  `Authorization: Bearer <jwt>` is sent. JWTs rotate **live** via
  `setAuthToken()` (mutates the SDK instance in place; no re-summon needed).

A cloud↔local switch can rebuild the SDK instance without re-summoning the bot.

### Multi-provider LLM factory

Anthropic (incl. the cloud proxy) is the default, but the brain supports a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sei-studio/sei](https://github.com/sei-studio/sei) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
