---
trigger: always_on
description: **Patrick is an agent-first patent-prosecution assistant. Open source, private by design, local-first.** Everything lives in the attorney's own folder, in open formats (`.docx`, `.pdf`), readable without the app. Zero lock-in, zero hidden state.
---

# Patrick — Agent Instructions & Project Guidelines

**Patrick is an agent-first patent-prosecution assistant. Open source, private by design, local-first.** Everything lives in the attorney's own folder, in open formats (`.docx`, `.pdf`), readable without the app. Zero lock-in, zero hidden state.

## Core Identity & Architecture
- **Product:** Agent-first patent-prosecution assistant (Tauri desktop app).
- **Core Philosophy:** Open-source (Apache-2.0), private by design, local-first. Zero lock-in, zero hidden state.
- **File Handling:** Read/write directly to native `.docx` (as tracked changes) and `.pdf` in the user's existing local folders. No proprietary databases.
- **AI & Data:** Bring-your-own-keys (Anthropic, OpenAI, Google, Vercel AI Gateway). Zero server uploads.
- **Transparency:** The system prompts, chain-of-thought, tool calls, context, and token costs must be fully visible to the user.
- **Voice**: Plain, Precise & Honest. Use factual, confident and careful language. Say exactly what Patrick does without hype or overselling whilst acknowledging limitations. 

## Monorepo

```
apps/
  frontend/   React 19 + Vite (rolldown) + Tailwind v4 + shadcn — the UI (webview for desktop)
  api/        Hono on Bun — local backend (compiles to a Tauri sidecar binary)
  desktop/    Tauri wrapper (webview = frontend, sidecar = api)
  site/       Next.js marketing + docs site (apps.patrick…) — alpha messaging, download, an own-MDX docs system
packages/
  shared/     types, model catalog, prompt token catalog, in-app docs (generated) — imported by frontend + api
  ui/          @patrick/ui — the shared design system: shadcn primitives (`components/*`) + cn + the stone/emerald tokens (`src/theme.css`), consumed by frontend + site (source-only)
  law/         @patrick/law — the EP law dataset + retrieval: EPC, EPO Guidelines, PCT-EPO Guidelines, Case Law of the Boards. Verbatim recall + find-the-law search source
  benchmarking/ a standalone grounding benchmark
scripts/        gen-patrick-docs.ts → packages/shared/src/patrick-docs.generated.ts (the agent's `patrick_help` corpus)
e2e/fixtures/   real-world .docx test fixtures for the headless redlining suites (run `bun test` from the repo root)
spikes/         throwaway proof-of-concept scripts (lint/knip-exempt, run by hand with bun)
living-docs/    transient per-task plans, deleted when the work ships (not durable docs)
SCRATCHPAD.md   is the **committed engineering backlog** — deferred bugs, parked refactors, and technical follow-ups surfaced during work; put durable engineering deferrals there (transient `living-docs/`).
```

**pnpm workspace**; configs (`pnpm-workspace.yaml`, `biome.json`, `tsconfig.base.json`, `knip.config.ts`, `bunfig.toml`) live at the root. A hosted/cloud app is the main future slot.

## Stack

React 19 + Vite + Tailwind v4 + shadcn (stone/emerald) · TanStack Router (file-based) + TanStack Query · @ansonlai/docx-redline-js (headless OOXML tracked changes, pinned + wrapped) · Hono on Bun · AI SDK v7 + `@ai-sdk/react` (Anthropic / OpenAI / Google / Gateway, **BYOK**) · Next.js (`apps/site`) · pnpm · Biome · TS strict · Streamdown (chat markdown) · `bun:test` (runner).

## Headless docx redlining (THE editing model)

Patrick's editing engine is `@ansonlai/docx-redline-js` (MIT, pinned to a commit), consumed through ONE adapter: **`apps/api/src/lib/docx/redline.ts` is the only import point** — everything else goes through it. An edit is a **paragraph-scoped reconciliation**: the agent names a paragraph's current text and its full revised text; the engine word-level-diffs them into a minimal native redline (`w:ins`/`w:del`). Untouched XML stays byte-identical — fidelity by construction, no whole-file round-trip.

**The adapter's guards are load-bearing (each one was a measured failure, not a hypothesis — don't bypass the adapter):**
- The engine's paragraph matcher fuzzy-falls-back (a missing target can redline the WRONG paragraph) → the adapter resolves the paragraph itself: exact, unambiguous match on the as-read text, full paragraph text handed to the engine.
- Re-redlining an already-redlined paragraph double-applies → the adapter **supersedes** its own pending revisions first (DOM surgery — safe because Patrick only ever emits plain `w:ins`/`w:del`); a paragraph's redline is always original → latest.
- Every edit is **verified before the write**, strictly at the edited position; a failure never mutates the file. Paragraphs carrying the ATTORNEY's own pending tracked changes are refused (comment instead) — editing through them would absorb their authorship.
- The engine stamps ghost formatting revisions (`w:rPrChange`) on rebuilt runs → stripped on every write. `numberingXml` from list-shaped rewrites is merged. Text-box content (`w:txbxContent`, `mc:Fallback`) is skipped everywhere — invisible, never duplicated.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mhurhangee/patrick](https://github.com/mhurhangee/patrick) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
