---
trigger: always_on
description: Guidance for AI coding agents working in **VibeOS** — an AI-hallucination-driven
---

# AGENTS.md

Guidance for AI coding agents working in **VibeOS** — an AI-hallucination-driven
operating system in the browser. Except for the core runtime, every window's UI
is generated in real time by AI via a pluggable provider (CodeBuddy / Claude Code
/ Codex / OpenRouter).

## Architecture (read this first)

The CLI-based providers (CodeBuddy, Claude Code, Codex) are driven **directly via
subprocess — no vendor SDKs** — so the AI layer is **backend-only — it cannot run
in the browser.** VibeOS is therefore:

```
packages/shared   protocol + domain types shared by both sides (@vibeos/shared)
apps/backend      Bun HTTP + WebSocket server: kernel/boot, provider manager, model
                  policy, prompt assembler, agent scheduler, syscall interpreter,
                  bun:sqlite repositories
apps/frontend     Vite + React 19 + Tailwind 4 + Zustand (custom token-based
                  design system + skins; NO component kit): desktop shell, window
                  manager, AI-HTML surface, context menus
```

- **SQLite is the single source of truth.** Frontend Zustand stores only mirror
  it; user intents always round-trip through the WebSocket.
- Transport: one WebSocket. Protocol is versioned envelopes with `c2s.*` /
  `s2c.*` message unions in `packages/shared/src/protocol`.
- **AI provider seam.** All model access goes through `ai/providers/` (`AiProvider`:
  `run()` + `discoverModels()`). `SdkManager.run()` resolves the role's model policy +
  localized system prompt, then delegates to the active provider — agents never touch a
  provider directly. CodeBuddy + Claude Code share `cli/AnthropicCliProvider` (both speak
  Anthropic stream-json via `<bin> -p --output-format stream-json`); Codex drives
  `codex exec --json`; OpenRouter is an HTTP provider (Vercel AI SDK). Active provider:
  Settings → env `VIBEOS_AI_PROVIDER` → `DEFAULT_PROVIDER` (claude); at boot, providers
  whose CLI isn't on PATH (`availableProviderIds()`) are skipped and the choice falls back
  to an available one (persisted). **UI generation is stateless** — each op is a fresh
  conversation (no session resume); the full current UI is sent as context every time
  (`[CURRENT UI]` in `PromptAssembler`, capped by `VIBEOS_SNAPSHOT_BUDGET`, 0 = no cap).
  Cost is taken from the Claude CLI's reported figure, else estimated from tokens
  (`ai/pricing.ts`) so codebuddy / codex / openrouter still show cost.
- **Skins.** `Settings.skin` (`devdock` | `xp` | `aqua`) sets `data-skin` on `<html>`;
  skins are pure CSS over the design tokens + `.vibe-*` chrome hooks, so the OS chrome
  AND the AI content re-skin live. The agent is **not** told the skin — keep generated
  HTML skin-neutral (token-based) so any app re-skins on switch.
- **i18n (zh / en).** `Settings.locale` drives both the native UI (frontend dictionary in
  `lib/i18n.ts`, `useT()`) and generated content (`localeDirective()` appended to every
  system prompt in `SdkManager`). Undefined locale ⇒ frontend follows the browser and
  persists the choice on first boot. Localize new chrome strings via the dictionary —
  never hardcode user-facing text.

## Commands

```bash
bun install
bun run dev          # backend (:7720) + frontend (:7730), via scripts/dev.ts
bun run dev:backend  # backend only
bun run dev:frontend # frontend only
bun run build        # production frontend build
bun run typecheck    # typecheck all three packages
```

Offline / no-model mode: `VIBEOS_AI_STUB=1 bun run dev` (deterministic stub UI).
Useful env: `PORT`, `VIBEOS_DB_PATH`, `VIBEOS_AGENTS_DISABLED=1`, `VIBEOS_LOG_LEVEL=debug`.

### Sandbox caveat (important)

This environment injects a broken `NODE_OPTIONS` preload that crashes any
`node`-spawned tooling (tsc, vite, the CLI). Always strip it:

- Typecheck a package: `NODE_OPTIONS= node_modules/.bin/tsc -p <tsconfig> --noEmit`
- `scripts/dev.ts` and `apps/backend/src/ai/providers/cli/env.ts` (`cliEnv`) strip
  `NODE_OPTIONS` when it points at `$bunfs` — the node-based CLIs (claude/codebuddy)
  would otherwise crash on startup. The CLIs are spawned directly (their own shebang),
  not under bun.

## Core mechanics (don't break these)

- **AI render modes**: the OS decides a baseline before calling the AI
  (`PromptAssembler.decideRenderMode` → `force-full` | `prefer-incremental`),
  tells the model, then `streamParser.parseAiOutput` finalizes by what came back:
  HTML containing only `data-vibeos-region` blocks → incremental patch
  (`mode:"regions"`), otherwise full replace (`mode:"full"`). Region extraction
  is depth-aware (handles nested elements) in both `streamParser.ts` and
  `agents/regionMerge.ts` — do not "simplify" it back to a single regex.
- **Per-window scheduling** (`agents/UiGenerationAgent.ts`): different windows
  run in parallel; within one window a new action **preempts** (aborts) the
  in-flight one ("latest wins"). Generation is stateless, so a preempt just
  aborts — there's no session to resume.
- **Event delegation** (`hooks/useDelegatedEvents.ts`): AI HTML never runs code.
  Clicks/submits/changes on `[data-vibeos-action]` become `c2s.op`. Clicks on
  editable inputs are passed through natively (never trigger generation). Forms

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [benis-me/VibeOS](https://github.com/benis-me/VibeOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
