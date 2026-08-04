---
trigger: always_on
description: Electron desktop **single-agent coding harness** (React 19, TypeScript, electron-vite) mirroring Codex in the Opus 4.5 era. Routes per-model to **thirty-two built-in providers** — frontier labs, public hosts and aggregators, regional specialists, local runtimes, and self-hosted gateways — plus unlimited user-defined **custom OpenAI-compatible endpoints** (`settings.json.customProviders`). The built-ins include DeepSeek, Google AI, DashScope, OpenRouter, Zhipu, OpenAI, Anthropic, xAI, Mistral, Mo
---

# Lamprey Harness — Codex Instructions

## What This Is
Electron desktop **single-agent coding harness** (React 19, TypeScript, electron-vite) mirroring Codex in the Opus 4.5 era. Routes per-model to **thirty-two built-in providers** — frontier labs, public hosts and aggregators, regional specialists, local runtimes, and self-hosted gateways — plus unlimited user-defined **custom OpenAI-compatible endpoints** (`settings.json.customProviders`). The built-ins include DeepSeek, Google AI, DashScope, OpenRouter, Zhipu, OpenAI, Anthropic, xAI, Mistral, Moonshot, Groq, Together, Fireworks, Cerebras, Hugging Face, AIHubMix, FreeLLMAPI, Cohere, MiniMax, NVIDIA, GitHub Models, SambaNova, SiliconFlow, Reka, SEA-LION, DeepInfra, Hyperbolic, Perplexity, Sarvam, Inception, and keyless Ollama + LM Studio. The model can fan out via the `multi_agent_run` tool (Task-tool analog); the always-on Planner→Coder→Reviewer pipeline was excised in the Unburdening Phase (v0.14.0).

## Architecture quick-pointers
- Provider registry + dispatch: `electron/services/providers/registry.ts` — `MODEL_CATALOG`, `chatStream`, `chatOnce`, `validateProviderKey`, `RETIRED_MODEL_MAP`. Adding a model = append to `MODEL_CATALOG`; Custom Models from settings.json are consulted by `resolveModel` (JM-11).
- Chat turn: `electron/ipc/chat.ts` — `chat:send` → `runHeadlessTurn` → `runChatRound` (also the seam loops/wake-ups run turns through). System prompts in `electron/services/system-prompt-builder.ts` (`AGENT_ROLE_PROMPTS`/`buildAgentSystemPrompt` survive for the `multi_agent_run` tool only).
- Multi-provider keychain: `electron/services/keychain.ts` keyed by provider id — any of the 32 built-ins or a custom endpoint id (+ `web_search:*` namespace for search providers). Provider-id authority is the registry (`resolveProviderDescriptor` / `isKnownProvider` / `listAllProviders`); the main + renderer `ProviderId` unions are parity-locked by `provider-parity.test.ts`. IPC: `settings:saveProviderKey` / `:test` / `:delete` / `:list`. Writes are atomic (temp+rename) via `electron/services/atomic-json.ts` (JM-13).
- UI surfaces: `src/components/settings/ApiKeySettings.tsx` (multi-provider list), `chat/ChatInput.tsx`, right-panel pills in `src/components/artifacts/RightPanelHome.tsx`.

## Current State
- **Direct-provider routing hotfix:** released 2026-07-19 as **v0.27.1**. All 70 pinned models route through their named provider and independently stored key; `MODEL_CATALOG` contains zero OpenRouter-brokered aliases. OpenRouter remains an explicit opt-in provider for live catalog imports only. Settings key saves synchronize the chat key-state store, the model menu refreshes keys whenever it opens, and a Moonshot key unlocks direct Kimi K3 without restart. The model menu is viewport-bounded and scrollable with the active entry revealed. Retired broker aliases map to direct equivalents; unsupported stale defaults fall back to a known catalog model.
- **Provider Discovery Expansion v2 (PX2-0–PX2-9):** complete (2026-07-19, **v0.27.0**) — 32 built-in providers and 79 pinned chat models. Adds API-key acceptance for AIHubMix, FreeLLMAPI, Cohere, MiniMax, NVIDIA, GitHub Models, SambaNova, SiliconFlow, Reka, SEA-LION, DeepInfra, Hyperbolic, Perplexity, Sarvam, and Inception Labs; refreshes current Kimi, Grok 4.20, and direct Gemma 4 rosters; adds descriptor-owned catalog normalization, filtered collision-safe bulk import, an editable FreeLLMAPI address, and MiniMax reasoning preservation. Imported capabilities default off until proven. Live authentication remains pending for providers whose keys were not available during the release. See `PLANNING/LAMPREY_PROVIDER_DISCOVERY_EXPANSION_PLAN.md` and `PLANNING/PX2_BASELINE.md`.
- **Prompts 1–20 + 21 + multi-provider revision**: complete (see `memory/project_build_status.md` for per-prompt commit SHAs).
- **RAG add-on (R1–R14)**: complete, audited, hardened (see DEVLOG 2026-06-03 audit entry).
- **Parity Phase (36 prompts + Integration H1–H6)**: complete — see `PLANNING/LAMPREY_PARITY_PLAN.md` and the H1–H6 wrap-up entry in `DEVLOG.md` (2026-06-04).
- **Fluidity Phase (J1–J11)**: complete (2026-06-04) — micro-interaction parity with Codex. Merged to `main` as commit `2691730`. See `PLANNING/LAMPREY_FLUIDITY_PLAN.md` and the per-prompt + phase-complete entries in `DEVLOG.md`. Eleven prompts shipped on `feat/fluidity-phase`: ESC + ↑ history, Shift+Tab mode cycle, @file mention, # memory shortcut, inline approval chips, tool-card auto-collapse, inline subagents, status-line context%, notification consolidation, path:line autolinking, right-panel default-collapsed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [USS-Parks/Lamprey-Harness](https://github.com/USS-Parks/Lamprey-Harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
