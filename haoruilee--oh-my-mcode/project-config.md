---
trigger: always_on
description: **IMPORTANT:** This file is the source of truth for agent instructions. Update it (or a nested `AGENTS.md`) when adding guidance. Do not add a parallel `CLAUDE.md` / Cursor rules dump.
---

# AGENTS.md

**IMPORTANT:** This file is the source of truth for agent instructions. Update it (or a nested `AGENTS.md`) when adding guidance. Do not add a parallel `CLAUDE.md` / Cursor rules dump.

## Commands

| Gate | Command |
| --- | --- |
| hermetic | `npm test` |
| hermetic eval | `npm run eval` (fixture harness; not live `mcode`) |
| CI | `.github/workflows/hermetic.yml` runs the two hermetic gates on push/PR to `main`. No live `mcode`, no MiniMax secrets. |
| live rematch | `oh-my-mcode plan` or `oh-my-mcode max` on a **copy** of `test/fixtures/hello-pkg` |

This repo uses **npm**. Do **not** use bun or pnpm — those rewrite the lockfile. Do not run a production publish or `npm publish` in an agent session.

Contributor operating manual for **oh-my-mcode**. Not a skill-pack blurb. This file does not become the orchestrator.

North star: **Not more agents. Ship with evidence. The TypeScript harness owns the loop. `mcode` is the host.**

## Default context

This repo is a **verified-delivery harness** that drives `mcode exec`. The plugin / TUI is a window. `src/orchestrator.ts` owns verify, repair, resume, and evidence. CLI and MCP share one core (`src/harness.ts` `submit`).

Unless the user says otherwise, assume work is this package — not a new host product, not a second runtime.

**Terminology** — when the user says **agent**, they mean **this product** (the MiniMax host process + our harness), not you (the assistant in this session).

| Term | Means |
| --- | --- |
| **max** | Hero command `oh-my-mcode max` (alias `omm`). Owns DISCOVER → PLAN → PLAN_REVIEW → EXECUTE → VERIFY → ACCEPT. |
| **plan** | `oh-my-mcode plan`. Stops at PLAN_REVIEW. Does **not** Accept. |
| **worker** | One role contract in `agents/*.md` for the **same** MiniMax host process. One `mcode exec`. No grandchildren. |
| **mcode** | Host binary (`@minimax-ai/code`). We do not wrap it as `mmx` / `mavis`. |
| **run store** | `<workspace>/.minimax/runs/<run_id>/` — `run.json`, `plan.md`, `tasks.json`, `events.jsonl`, `evidence/`, `findings.json`, `summary.md`. |

Host already has explore / plan / team. Role files do **not** register new host agents. Do not pretend we spawned Sisyphus.

## Layout

| Path | What it is |
| --- | --- |
| `src/orchestrator.ts` | Phase machine. `plan` stops at PLAN_REVIEW. `max` continues to ACCEPT. |
| `dist/` | Generated `tsc` output. Edit `src/`, then `npm test`. Do not hand-edit compiled JS. |
| `src/harness.ts` | One core: `submit` / subscribe / bind. CLI and MCP call this. |
| `src/subagent.ts` | One role worker per `mcode exec`. Depth ≥ 1 throws. |
| `src/yield.ts` | `schemaMode=strict`. Parent parses `exec.result.answer` / assistant JSON / `structuredOutput.data`. |
| `src/mcode.ts` | Host argv + exit map. Default omits `--output-schema`. |
| `agents/*.md` | Five role contracts for the same host process. |
| `skills/*/SKILL.md` | TUI phrasing (`max` / `plan` / `verify` / …). Not a second loop. |
| `schemas/*.schema.json` | On-disk schemas. TypeScript validates. |
| `docs/host-reality.md` | Observed mcode 0.2.1 contract. Read this; do not invent. |
| `docs/design-check.md` | Eight questions after each cut. Fail → change the cut. |
| `docs/harness.md` | Codex-as-platform map. |
| `test/host-contract.test.mjs` | Locked host argv / stream / yield contract. |
| `test/fixtures/hello-pkg` | Live rematch / plan-max fixture (`hello()` imported, `placeholder()` exported). Keep plan/max tests pointed here. |
| `test/fixtures/hello-repair` | Two-export fixture (`hello()` + `greet()` imported, only `placeholder()` exported). Hermetic `npm test` in the fixture must fail. |
| `test/fixtures/fake-mcode.mjs` | Hermetic stub. **Not** live QA. |
| `examples/AGENTS.max-mode.md` | Opt-in template for a **user** product repo. |

## Max Mode

An agent in **this** repo, or in a user project that installed the plugin, must drive the real loop — not re-implement phases in prose.

```bash
oh-my-mcode max "<goal>" --workspace <project> --permission smart
# alias:
omm max "<goal>" --workspace <project> --permission smart
```

| Command | Stops at | Accepts? | Edits product code? |
| --- | --- | --- | --- |
| `oh-my-mcode plan "<goal>"` | PLAN_REVIEW | No | No (run artifacts only) |
| `oh-my-mcode max "<goal>"` | ACCEPT | Only via verifier + evidence files | Yes, in EXECUTE |
| TUI phrasing `max mode: …` | same as `max` | same | same |
| TUI phrasing `make a verified plan` | PLAN_REVIEW | No | No |

`--workspace` is the project being changed (default: cwd). `--permission smart` is the usual builder mode (`ask|smart|full|off`). `--no-llm-verify` skips the optional read-only LLM judge; deterministic verify still runs.

Run store: `<workspace>/.minimax/runs/<run_id>/`. If there is no folder, the loop did not run.

User-project copy-paste: [`examples/AGENTS.max-mode.md`](examples/AGENTS.max-mode.md). `oh-my-mcode install` does **not** write that file over a project `AGENTS.md`.

## Workers

Five roles. Same host process. One exec each. Point at the contract file — do not invent a sixth.

| Who | File | Mode | May | Must not | Yield |
| --- | --- | --- | --- | --- | --- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [haoruilee/oh-my-mcode](https://github.com/haoruilee/oh-my-mcode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
