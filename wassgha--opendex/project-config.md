---
trigger: always_on
description: OpenDex is an **Electron** desktop app (electron-vite + React + Tailwind v4). It is a voice-first agentic harness, generalized from a Next.js demo.
---

# OpenDex — agent notes

OpenDex is an **Electron** desktop app (electron-vite + React + Tailwind v4). It is a voice-first agentic harness, generalized from a Next.js demo.

## Process model (important)
- **Main process** (`src/main/`) — Node. Owns the agent loop (`agent/chat.ts`, Vercel AI SDK `streamText`), TTS (`tts/elevenlabs.ts`), API keys, and all IPC handlers (`index.ts`). **Secrets never reach the renderer.**
- **Preload** (`src/preload/`) — `contextBridge` exposes a minimal typed `window.opendex` API (`chat`, `synthesize`). Ambient global in `opendex.d.ts`.
- **Renderer** (`src/renderer/src/`) — React UI + the voice state machine (`lib/dex/use-dex.ts`, hook `useDex`). Talks to main only through `window.opendex`. `@/` aliases `src/renderer/src`. (Note: no `Jarvis` identifiers in code — "Jarvis" is only the name of one theme.)

## Conventions
- Add a new IPC channel: declare it in `src/main/ipc/channels.ts`, handle it in `src/main/index.ts`, expose it in `src/preload/index.ts`, type it in `opendex.d.ts`.
- Main-process runtime deps go in `dependencies` (electron-vite externalizes them; electron-builder ships them). Renderer-only libs (react, etc.) can be `devDependencies` — Vite bundles them.
- Typecheck with `pnpm typecheck`. Smoke-test the agent without Electron: `pnpm smoke:chat [briefing]`.
- Build: `pnpm build` (→ `out/`); run built app: `pnpm start`; package: `pnpm dist`.
- **macOS signing — `build/entitlements.mac.plist` must contain NO XML comments.** Apple's entitlements parser (`AMFIUnserializeXML`) is stricter than a normal plist parser and rejects `<!-- … -->`, failing codesign with `AMFIUnserializeXML: syntax error` (the reported line is approximate / the next element). Use canonical `<true/>` (no space) and keep rationale out of the file. Both `entitlements` and `entitlementsInherit` point at this one file. Entitlement rationale: `allow-jit` + `allow-unsigned-executable-memory` for the WASM voice engines (Vosk/ORT) under hardened runtime; `disable-library-validation` to load the separately-signed nut-js `libnut`; `device.audio-input` mic; `network.client/server`, `files.user-selected.read-write` for the agent/TTS/STT and the Open skill.

## Config (Phase 2)
- `src/main/config/schema.ts` — `OpenDexConfig` shape + `DEFAULT_CONFIG` + `mergeConfig`. Shared types only with the renderer (imported type-only across the process boundary).
- `src/main/config/store.ts` — hand-rolled store: `config.json` (prefs) + `secrets.json` (keys encrypted via `safeStorage`) in `userData`. `applyToEnv()` pushes config + decrypted secrets into `process.env` so agent/TTS read them as before; dev `.env` is a fallback for unset secrets. **Secret values never leave main** — IPC returns only presence booleans (`PublicConfig`).
- Greeting/persona/model are resolved per-turn in the chat handler via `buildSystemPrompt({config, briefing})` (`agent/system-prompt.ts`); greeting modes: `example` (CoreViz) | `custom` | `none`.
- Renderer: `lib/use-config.ts` loads/updates config; `App.tsx` gates on `onboarding.completed` → `OnboardingWizard` else `MainExperience`. TTS engine chosen via `lib/dex/speech-engine.ts` (ElevenLabs vs `SystemSpeechEngine`).

## LLM providers (pluggable)
- `config.llm = { provider, model }`. `provider`: `apple` (free on-device, macOS/Apple Silicon, native `@meridius-labs/apple-on-device-ai`) | `openai` | `anthropic` (both BYOK) | `gateway` (Vercel AI Gateway — one key, any provider; bare model id) | `opendex` (hosted subscription — **reserved, not implemented**: `auth: "account"`, renders disabled "coming soon").
- `src/main/config/llm-providers.ts` — pure, dependency-free metadata (`LLM_PROVIDERS`: label, `kind` local/byok/managed, `auth` none/key/account, `secretName`, curated `models`, `keyUrl`, `platforms`, `comingSoon`, `supportsTools`). Imported as a **value** by both main (resolver) and renderer (picker). (Skill metadata is similarly shared, sourced once per skill from `src/skills/<name>/meta.ts`.)
- `src/main/agent/llm/resolve-model.ts` — `resolveModel(config)` maps provider→AI SDK model (or bare string for gateway); throws a user-facing reason for unset keys / unavailable Apple / the not-built subscription, surfaced as a spoken apology by the chat handler. `chatStart` calls it per-turn (`index.ts`). Apple is **dynamic-imported** (native Swift/Rust, darwin-only) so non-mac builds never load it; `checkAppleAvailability()` (via `llm:apple-availability` IPC) gates the picker.
- New secret `ANTHROPIC_API_KEY`. `DEFAULT_CONFIG.llm.provider = "gateway"` so pre-provider configs (only `llm.model`) keep working after merge.
- Renderer: `components/llm/provider-picker.tsx` (`ProviderPicker` + `useAppleAvailability` + `isProviderReady`/`defaultModelFor`) — shared by onboarding (no default; "Continue" gated until a provider is chosen + usable) and Settings → Language model. Apple shows disabled-with-reason on Macs, hidden entirely off-Mac.

## Themes (Phase 3) — full-interface

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wassgha/opendex](https://github.com/wassgha/opendex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
