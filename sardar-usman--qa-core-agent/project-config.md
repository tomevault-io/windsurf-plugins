---
trigger: always_on
description: This file is loaded automatically into Claude Code's context at the start of every conversation in this project. Anything here is "free" context — no need to re-explain.
---

# QA-Core — project rules for Claude

This file is loaded automatically into Claude Code's context at the start of every conversation in this project. Anything here is "free" context — no need to re-explain.

---

## Project identity

QA-Core is an autonomous QA agent that drives a real browser, reviews its own work, and emits a complete Playwright test framework where every line passes 5 independent executions in fresh browser contexts before the file is written.

Built on:

- **Claude** (via Anthropic SDK) — Opus 4.7 for exploration, Sonnet 4.6 for review/generate, Haiku 4.5 for planning (in-run selector recovery is deterministic, no model; spec healing is the qa-core-heal package)
- **Playwright** + TypeScript — the test framework and the live browser the agent drives
- **OpenClaw** — the persona, channel routing, and slash-command layer
- **MCP** — for native integration in Claude Desktop, Cursor, Cline, etc.

It is also Muhammad Usman's flagship portfolio project, positioning him as an AI Test Engineering Lead. Output quality matters because it ships publicly.

---

## The five-stage pipeline (do not break this order or skip stages)

| Stage | Module | Cost | Job |
|---|---|---|---|
| 1. Planner | `src/agent/planner.ts` | Haiku ~$0.001 | One page snapshot → numbered scenario list |
| 2. Explorer | `src/agent/runtime.ts` + `src/agent/tools.ts` | Opus ~$0.10–$0.30 | Drives Chromium via tool-use, records verified trace |
| 3. Critic | `src/agent/critic.ts` | Sonnet ~$0.005 | Per-scenario verdict — `pass` / `rework` / `reject`; non-pass dropped before replay |
| 4. Reality-Check Replay | `src/agent/replay.ts` | Zero LLM | Re-runs each scenario once in a fresh context, drops failures |
| 5. Stability Iteration | `src/agent/stability.ts` | Zero LLM | Re-runs each survivor 3× more, drops flakes |

Output is then transcribed by `pom.ts` (default — full POM framework) or `transcriber.ts` (`--no-pom` — single inline file).

---

## Key invariants — every fix must preserve these

1. **Every emitted scenario contains at least one assertion.** Enforced in `tools.ts` (`end_scenario`, `finish`) and `runtime.ts` (force-push guard at the end of the explore loop).
2. **The selector cascade only claims a level when it resolves to exactly one element.** When ambiguous, the `ambiguous` flag is set on `SelectorRecord` and `.first()` is honestly emitted by both replay and the transcriber.
3. **`toHaveCount` uses the multi-match locator, NOT the `.first()`-wrapped one.** Uses `baseLocator` from `replay.ts` and strips `ambiguous` from the recorded step.
4. **Every scenario starts from a clean state.** `begin_scenario` clears cookies + storage. The transcribed spec emits a matching `beforeEach`.
5. **All `page.evaluate` calls work under `tsx`.** `installEvalShim(ctx)` from `src/agent/eval-shim.ts` must be called on every new browser context, immediately after `browser.newContext()`. Without it, `tsx`'s `__name` keepNames helper crashes the evaluated function.
6. **The Planner parser accepts the v3.1 `[feature][category]` format AND all 4 legacy variants.** Don't tighten the regex without re-running `scripts/smoke-planner-parse.ts`.
7. **Dashboard per-site math divides by `runsWithPass`, not total runs** ([qa-core-ui.html#L4044](qa-core-ui.html)). Don't revert this.
8. **Framework dir name is `<brand>-automation-framework`.** Brand is the hostname with `www.` stripped and the TLD dropped (e.g. `www.saucedemo.com` → `saucedemo`). Use the `frameworkDirName()` / `brandSlug()` helpers exported from `src/agent/scaffold.ts`; do not hand-roll.
9. **Page objects are grouped by `feature`, not by URL pathname.** The Planner tags each scenario with a feature (e.g. `login`, `cart`). `pom.ts` emits one `pages/<feature>-page.{ts,js}` (kebab-case filename, PascalCase class) per feature and one spec per feature at `tests/<feature>/<feature>.spec.{ts,js}`. The a11y audit lives at `tests/a11y/landing.a11y.spec.{ts,js}` (inside `tests/` so it's picked up by the default `testDir: './tests'` — no manual `--testDir` flag needed). Scenarios without a feature fall back to URL-pathname grouping (legacy traces).
10. **Both TypeScript AND JavaScript output paths must stay clean.** TS emits `playwright.config.ts` + `tsconfig.json` + TS syntax. JS emits `playwright.config.js` (CommonJS) + NO `tsconfig.json` + `// @ts-check` JSDoc helpers. The agent must produce a fully runnable project in either language — `cd <out> && npm install && npx playwright test` must work for both.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sardar-usman/qa-core-agent](https://github.com/sardar-usman/qa-core-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
