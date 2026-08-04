---
trigger: always_on
description: PromptSoul is a local, self-hosted Next.js AI NPC prototype. The Node backend returns a short character reply plus an emotion label; the browser maps that emotion to a Live2D motion. Its motion workshop can also turn a natural-language prompt into a strictly constrained motion specification.
---

# AGENTS.md — PromptSoul Agent Guide

PromptSoul is a local, self-hosted Next.js AI NPC prototype. The Node backend returns a short character reply plus an emotion label; the browser maps that emotion to a Live2D motion. Its motion workshop can also turn a natural-language prompt into a strictly constrained motion specification.

Read this file and `README.md` before changing the project. Preserve completed behavior instead of rebuilding it.

## Architecture

- `app/`, `components/`: Next.js App Router UI and Node Route Handlers.
- `assets/styles.css`, `assets/app.js`: responsive Live2D rendering, interaction, chat and emotion mapping. The legacy runtime is intentionally loaded by `components/legacy-runtime.tsx` to preserve the proven renderer.
- `npc.config.json`: public character copy, suggestions, persona and model attribution.
- `lib/server/provider-*`: OpenAI-compatible Chat Completions client and process-memory settings.
- `lib/server/chat-service.ts`: deterministic demo replies plus provider response parsing.
- `lib/server/motion-*`: safe prompt-motion compiler, authoring, generation and independent validation.
- `lib/server/model-*`: model import, safe ZIP handling, Hiyori setup and model analysis.
- `scripts/`: Node/TypeScript CLI entry points. The project has no Python dependency.

The chat API returns `{reply, emotion, mode}`. Allowed emotions are `happy`, `wink`, `nod`, `thinking`, `surprised`, `shy`, `shakehead`, and `neutral`.

The preferred generated group is always `PromptSoul`. `Action` is only a read/play compatibility fallback in the browser.

## API Key boundary

- The settings UI may accept an API Key, but it must submit it only to the same-origin local Node backend.
- Runtime keys live only in Node process memory. Never write them to React state longer than needed, browser storage, cookies, files, config, logs, responses or Git.
- Public provider/status endpoints may expose mode, source, model and API base, but never the key.
- Keep environment variables (`NPC_API_KEY` or `OPENAI_API_KEY`) as the recommended production configuration.
- Provider mutations must remain loopback-only, same-origin JSON requests. Do not expose this development server directly to the public internet.

## Typical model workflow

When the user asks to add motions to a model, work autonomously:

1. Locate a supplied ZIP/folder, or search `local-assets/` and the repository for `*.model3.json`. Ask only if no model can be found.
2. Import it with `npm run setup:model -- <zip-or-folder>`. This creates a working copy under `models/` and an ignored `model.config.json`. For the official Hiyori demo, first read the linked Live2D terms and run `npm run setup:demo -- --accept-license`.
3. Always run `npm run analyze:model` before designing motions for a new model.
4. Create or extend `motion-defs/<model-stem>.ts` using that model’s actual parameters. `motion-defs/hiyori_pro_t11.ts` is a format example, not a parameter template.
5. Run `npm run motions:generate`.
6. Fix issues until `npm run motions:validate` passes.
7. Run `npm run verify:browser` and visually inspect real rendered poses.
8. Start `npm run dev` and hand over <http://127.0.0.1:8765>.

## Repository layout

```text
app/                           UI and Node Route Handlers
components/                    React components
assets/                        Live2D browser runtime and CSS
lib/server/                    Server-only provider/chat/model/motion logic
scripts/                       TypeScript CLI entry points
tests-node/                    node:test suites
tools/verify_browser.sh        Chrome screenshot verification (Node server; no Python)
motion-defs/<model>.ts         Model-specific base motion definitions
motion-defs/generated/         Ignored validated AI specifications
npc.config.json                Public character/UI configuration
model.config.json              Ignored generated active-model pointer
local-assets/ , models/        Ignored licensed/generated model data
tmp-verify/                    Ignored browser screenshots
```

Everything under `models/` is generated. Never hand-edit `.motion3.json` or `model3.json`: edit the matching TypeScript definition and regenerate, or use the workshop, which persists a validated spec under `motion-defs/generated/`.

## Motion workshop safety boundary

- The browser sends only `{prompt}` to `/api/motions/generate`. The API Key, model path, raw parameter IDs, curves and raw provider response stay server-side.
- The provider sees opaque controls plus semantic display names and normalized values. The compiler maps them back to the current model profile.
- Treat provider output as data only. Never add `eval`, generated JavaScript/TypeScript, provider-selected paths, shell execution or subprocesses to authoring.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [promptwhisper/promptsoul](https://github.com/promptwhisper/promptsoul) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
