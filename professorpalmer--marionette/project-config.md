---
trigger: always_on
description: Marionette is the product: a frontier/any pilot shell over a Puppetmaster
---

# AGENTS.md -- Marionette

Marionette is the product: a frontier/any pilot shell over a Puppetmaster
kernel. The `pmharness/` package is the research/eval rig that validated the
driver seam; it is not the shipping GUI contract.

## Product vs research

| Surface | Own it when… | Key modules |
|---|---|---|
| Product (lane B) | GUI/CLI pilot loop, tools, SSE, delegation | `harness/conversation.py` (facade) + mixins (`send_loop`, `busy_control`, …), `harness/api/*` HTTP peels, `harness/pilot.py` (`PilotTurn` / schema), `harness/tool_dispatch.py`, `harness/tool_discovery.py`, `harness/pilot_guards.py`, `harness/hash_edit.py` |
| Research rig | DriverIntent eval, scoring, Stage batteries | `pmharness/intent.py`, `pmharness/bridge.py`, `pmharness/drivers/`, `harness/session.py` (single-shot) |

Ownership rule: new pilot tools -> `pilot.py` schema + `tool_dispatch` /
`tool_discovery`; new orchestration -> Puppetmaster; do not grow
`conversation.py` with per-tool handlers.

## Conventions

- No emojis or decorative pictographs anywhere (code, docs, commits, output).
  Plain words only.
- stdlib-only for the harness/rig itself (urllib, sqlite, dataclasses).
  Puppetmaster is the single real dependency, installed editable from the local
  checkout.
- The `pmharness/intent.py` layer must stay PM-free and pure so it unit-tests
  fast and hermetically. Execution coupling lives only in `bridge.py`.
- Scoring is deterministic -- no LLM-as-judge. Every metric must be a function
  of (labeled task, raw driver text, execution result).
- Driver eval measures driving, not working: swarm intents execute on
  Puppetmaster's free local adapter for deterministic ground truth.
- Tests before claiming done: `.venv/bin/python -m pytest -q`. The offline E2E
  test drives real Puppetmaster and must stay green with zero API keys.
- Releases only from green CI: never push a release tag until the `tests`
  workflow is green for this git tree (3.9 floor, 3.11, Windows, frontend-build).
  If dest-into-main `merge^{tree}` equals the dest PR tree that already passed,
  tag immediately -- do not wait for main to run the same suite again. The
  release workflow must not re-run pytest; it only checks that a successful
  `tests` run exists for the tree, then publishes installers. Local tests
  pass on the dev interpreter only; CI is what proves the 3.9 floor.
- Never commit keys or `results/*.sqlite`.
- Git flow: day-to-day work is on `dev`. Feature PRs target `dev`. Ship by
  merging `dev` into `main` (dest contains `main` first), tagging when the
  dest-into-main PR `tests` matrix is green and `merge^{tree}` matches, then
  pushing `vX.Y.Z` on `main`. Never push product work or `pmedit-*` worker
  branches to `main` / origin scratch.

<!-- puppetmaster:rules:begin -->
<!-- managed by `puppetmaster install-rules`; delete this whole block to disable -->

# Puppetmaster orchestration

Puppetmaster is an MCP-based agent orchestrator with structured worker
swarms, durable SQLite state, tiered model routing, and zero-token
follow-ups via stored artifacts. When Puppetmaster's MCP server is
registered (`puppetmaster install-cursor-mcp` or
`puppetmaster install-codex-mcp`), the `puppetmaster_*` MCP tools are
available in this environment.

## Trigger convention (must obey)

When the user says **"Use Puppetmaster to …"**, **"PM this …"**, or
otherwise names Puppetmaster for a task, route that work through the
`puppetmaster_*` MCP tools — do not answer inline.

## Delegate-first gate (default path)

Before attempting multi-step work inline, start a Puppetmaster verb
(`puppetmaster_start_cursor_swarm`, `puppetmaster_start_swarm`,
`puppetmaster_start_implement`, or the matching sync verbs) when the
task is any of:

- Multi-file (3+ files) or cross-cutting refactor/migration
- An audit, review, or "find all X" search
- Work whose result will be reused later in this or a future session

Swarms and reviews run read-only analysis; building goes through
implement. Recall prior results with `puppetmaster_artifacts <job_id>`
at zero token cost.

Reach for a Puppetmaster verb **before** native broad search/exploration:
prefer `puppetmaster_codegraph_search` / `_context` over a repo-wide
`Grep`/`Glob`/`find`, and a swarm over the built-in `Task` tool, for any
multi-file investigation. When unsure whether a task qualifies, run the
classifier-backed gate — `puppetmaster_route_task` (or
`puppetmaster should-delegate "<prompt>"`) — which returns a delegate /
inline verdict and a suggested verb with zero LLM cost.

For deterministic enforcement, the user can install host hooks
(`puppetmaster install-hooks`) that inject this directive on prompt submit
and deny-redirect broad native exploration automatically. The kill switch
is `PUPPETMASTER_AUTO_INVOKE_DISABLED=1`.

## CodeGraph-first exploration (must obey)

CodeGraph is the default way to explore code — graph every directory you
interact with, then explore the graph instead of crawling the tree:

1. **Graph it first.** Before exploring any directory (the workspace root
   or a subtree you're diving into), check `puppetmaster_codegraph_status`;
   if it has no `.codegraph/`, run `puppetmaster_codegraph_init`
   (`index: true`) — it returns immediately and indexes in the background.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [professorpalmer/marionette](https://github.com/professorpalmer/marionette) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
