---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

WebGenie — a Manifest V3 Chrome extension that runs a planner + navigator LLM browser-automation loop entirely client-side. pnpm + turbo monorepo (`chrome-extension`, `pages/*`, `packages/*`). `docs/agent.md` is the long-form description of the agent; `docs/e2e.md` covers the live test harness.

## Commands

```bash
pnpm install                       # pnpm 9.15.1, node >= 22.12
pnpm build                         # clean + turbo ready + build -> ./dist (load unpacked)
pnpm dev                           # watch build, __DEV__=true, HMR reload
pnpm type-check                    # tsc --noEmit across all packages
pnpm lint / pnpm prettier          # lint runs eslint --fix

pnpm -F chrome-extension test                              # vitest (only package with tests)
pnpm -F chrome-extension exec vitest run src/background/agent/validation/__tests__/settling.test.ts
pnpm -F chrome-extension exec vitest run -t "test name"

pnpm e2e                                              # build + live core suite (Vertex AI via gcloud)
E2E_SUITE=security pnpm -F chrome-extension e2e       # core|complex|hitl|security|breadth|endurance|resilience|memory|all
E2E_ONLY=T1,C13 E2E_HEADLESS=1 pnpm -F chrome-extension e2e
E2E_ORACLE=1 E2E_SUITE=all pnpm -F chrome-extension e2e   # check the checkers, no model calls
```

There is no root `test` script and no `test` task in `turbo.json` — unit tests only run through the `chrome-extension` package. `pnpm e2e` drives the built extension in Chromium against real sites and local fixtures; options (`E2E_MODEL`, `E2E_PLANNER_MODEL`, `E2E_REPEAT`, token caps, baseline update) are listed in `docs/e2e.md`. Results (summary with Wilson intervals and health counters, per-task events/trace/screenshot) go to the gitignored `chrome-extension/e2e/results/`; tasks whose site is unreachable are `site_down` and excluded. `chrome-extension/e2e/mind2web/` runs and judges an Online-Mind2Web slice.

`turbo ready` must run before any build: `packages/*` emit their `dist/` there and `@extension/i18n` generates its typed message keys. The root `build`/`dev` scripts already do this; if you build a single package directly, run `pnpm -F <pkg> ready` first.

## Architecture

### Runtime topology
Three independent Vite builds land in one `dist/`: the background service worker (`chrome-extension/`, single IIFE bundle from `src/background/index.ts`), the React pages (`pages/side-panel`, `pages/options`), and the content script (`pages/content`, which draws the agent's on-page presence: the cursor, the halo on the element it is about to use, the typing caret, the goal chip, the status capsule and the ambient border, in one shadow-root overlay that takes no pointer events and hides itself while a screenshot is captured). `chrome-extension/manifest.js` is the manifest source of truth (generated at build time by `utils/plugins/make-manifest-plugin`); edit it, not `dist/manifest.json`.

The side panel talks to the background over a long-lived port named `side-panel-connection`. The command switch in `chrome-extension/src/background/index.ts` is the single entry point for everything the UI can trigger: `new_task`, `follow_up_task`, `cancel_task`/`pause_task`/`resume_task`, `human_response`, `reattach`, `resume_saved_task`, `discard_saved_task`, `screenshot`, `state`, `nohighlight`, `speech_to_text`, `replay`, `heartbeat`. Agent progress flows back as events emitted by `EventManager` and forwarded over the same port.

### The agent loop
`agent/executor.ts` owns one task. Per step it: decides whether to re-plan (`contracts/replan.ts` — initial, human needed, a sent-back plan (`completion_refused`), navigator-claimed completion, navigator error, unvalidated steps, stall, or the `planningInterval` cadence, which waits while every step since the last plan validated, up to twice the interval), builds the page state once, runs `PlannerAgent` (when re-planning) and `NavigatorAgent` with that state, and saves a checkpoint. The navigator never runs before a first successful plan. Completion is the *planner's* call; the navigator's `done` text is only the provisional answer. The executor sends a planner `done` back while the plan's `requirements` checklist has a pending item (at most twice), when an applied requirement's evidence is gone from the current page, when an answer of absence rests on an unread page, or when the answer carries nothing the task asked for (once each), and sends back a goal planned a third time without progress (from the fifth the planner must finish). `validation/done-evidence.ts` logs whether a `done` was backed by evidence next to the planner's verdict (it only skips the planner with `acceptEvidencedDone`, default off). `contracts/execution-router.ts` runs a bare "go to <url>" task without a planner call.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [derpx06/webgenie](https://github.com/derpx06/webgenie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
