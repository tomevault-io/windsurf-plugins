---
trigger: always_on
description: Guidance for working in this repo. Keep it short and accurate; update it when architecture changes.
---

# CLAUDE.md

Guidance for working in this repo. Keep it short and accurate; update it when architecture changes.

## What this is

**吟游手册 / REtavern-card-helper** — a visual helper for authoring [SillyTavern](https://github.com/SillyTavern/SillyTavern) character cards. Users fill in a step wizard (character设定, world book, first message, MVU status-bar variables, staged lorebook, live-stream chat) with AI assistance, then export a spec-compliant card as **PNG or JSON**.

All user data lives in the **browser (IndexedDB via Dexie)** — nothing is stored server-side. The server is only a **CORS proxy** for user-supplied AI API keys; it never stores keys.

## Commands

```bash
npm run dev         # Vite client + Hono proxy (port 3001) via concurrently
npm run typecheck   # tsc --noEmit  — MUST stay green
npm run lint        # eslint src   (0 errors expected; ~warnings tolerated)
npm test            # vitest run   (all tests must pass)
npm run build       # vite build → dist/
```

Always run `npm run typecheck` and `npm test` after changes. The test suite is the safety net for the card-export / MVU / lorebook logic.

## Stack & layout

- **React 19 + TypeScript + Vite + Tailwind v4**. Routing: `react-router-dom` (`src/App.tsx`), all routes lazy-loaded.
- **Hono** proxy: `server/` (Node entry `server/index.js`) locally; Cloudflare Worker in production. Deploy targets in [DEPLOY.md](DEPLOY.md) (Vercel / Docker / Node / Cloudflare).
- `src/pages/` — route pages. `WizardPage.tsx` orchestrates the wizard; each `StepX` is **lazy-loaded** (keeps the wizard's initial chunk small — do not convert these back to eager imports).
- `src/components/wizard/` — the step components. `src/components/novel-workshop/` — a self-contained "novel → card" feature module.
- `src/services/` — the core logic (see below). `src/hooks/` — wizard/AI state. `src/db/database.ts` — Dexie schema (`cards`, `chat_sessions`, `ai_settings`).
- `src/constants/` — `defaults.ts` (types + shared constants), `prompts.ts` (AI prompt templates only), `theme.ts`.
- `src/utils/` — cross-cutting helpers: `deep-clone.ts` (`deepClone`), `html.ts` (`escapeHtml`).

## Core services (high-value, well-tested — change carefully)

- `card-exporter.ts` — assembles a `WizardDraft` into a Tavern V2/V3 card and back (`assembleCard` / `cardToDraft`), PNG export, and status-bar/live-chat regex-script wiring.
- `mvu-builder.ts` — builds MVU status-bar variable schema/YAML/EJS from config.
- `staged-lorebook-builder.ts` — staged (阶段) lorebook entries; **exports the canonical EJS escapers** `escapeEjsSingleQuoted` / `escapeEjsDoubleQuoted`.
- `card-chat-optimizer.ts` — applies AI-proposed patches to a draft / raw card JSON.
- `png-service.ts` — PNG tEXt chunk read/write for embedding card JSON (`chara` V2, `ccv3` V3).
- `card-validator.ts`, `quality-checker.ts` — pre-export checks and quality scoring.
- `ai-json.ts` — parses JSON out of AI replies (`parseAIJson`, `stripMarkdownFences`). Zero deps; keep it that way so callers don't pull in the prompt constants. Note `ai-service.ts` deliberately does **not** use `stripMarkdownFences` — it needs lenient, unpaired fence stripping for truncation detection.
- `mvu-sim.ts` — MVU (MagVarUpdate) variable engine simulator for the test chat: parses `[InitVar]` / `setvar` initial values, replays `_.set/insert/delete/add` + `<JSONPatch>` commands per message, and substitutes `{{get|format_message_variable::}}` macros. Pure functions, zero deps. The module header documents every intentional deviation from the real runtime — **keep that list in sync** when changing behavior. Reference sources live in gitignored local clones (`magvarupdate/`, `js-slash-runner/`, `st-prompt-template/`).

## Conventions & gotchas

- **Single sources of truth — do not re-duplicate:**
  - EJS string escaping → `escapeEjsSingleQuoted` / `escapeEjsDoubleQuoted` from `services/staged-lorebook-builder.ts`.
  - HTML escaping (app-side rendering) → `escapeHtml(str, { quotes })` from `utils/html.ts`. (The `lcEsc` inside `live-chat-templates.ts` is *generated runtime code that ships inside the card* — leave it.)
  - Deep clone → `deepClone` from `utils/deep-clone.ts`.
  - Regex-script names → `REGEX_SCRIPT_NAMES` in `constants/defaults.ts` (`状态栏界面` / `直播间界面`). These strings are matched by name on import/validate/patch — never hard-code the literals.
  - **World book name** → `resolveBookName(draft)` in `constants/defaults.ts`. The exported `character_book.name`, `extensions.world`, and the `getWorldInfo("书名", …)` argument inside staged dispatcher entries must all come from it. ST's `loadWorldInfo` matches the name **exactly**; any divergence makes dispatcher entries fetch nothing — that is the root cause of the "阶段不切换" class of bugs. Export additionally runs `alignStagedDispatcherBookName` so dispatcher entries baked with an older name get rewritten.
- **EJS escaping is a security boundary**: user/AI text is embedded into generated EJS/JS templates. Any new embedding must go through the escapers above (neutralizes `%>`, quotes, line separators).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shankongjiu-dot/REtavern-card-helper](https://github.com/shankongjiu-dot/REtavern-card-helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
