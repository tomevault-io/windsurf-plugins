---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

MELRA is an agent-independent **autonomy kernel**: the layer an agent asks to
change the world through. Three layers, and the split is the whole design — the
LLM reasons, the harness manages the loop, MELRA owns the effect lifecycle.
MELRA begins where the tool call leaves the model loop. It owns effects, never
reasoning — nothing in `packages/` may call a model, and no decision in the
execution path may depend on model output.

**The feature test.** Before adding anything, ask: *would this feature still
make sense if the effect request came from ordinary deterministic software
rather than an LLM?* Authorization, idempotency, credential isolation, recovery,
verification, effect history, capabilities — yes, they belong here. Prompt
optimization, LLM memory, model selection, a planner, agent personality — no,
they belong to the harness above.

For every effect it does exactly nine things and nothing else: type it against a
strict schema, classify it, authorise it against policy, gate it on an exact
approval phrase, record it durably before anything runs, deduplicate it by
idempotency key, run it under a budget and cancel signal, verify it against
declared evidence, and receipt it. Work that is not one of those nine jobs — a
model router, a planner, a prompt library, semantic memory about the user —
belongs to the agent above and does not go in this repo.

MCP over stdio is one of several interfaces onto the same runtime (MCP stdio,
MCP over loopback HTTP, CLI, TypeScript SDK, Python SDK, read-only JSON API);
none of them is a shortcut past a stage of the pipeline. Eleven MCP tools sit in
front of five reference effect adapters (files, terminal, browser, computer, http)
plus
operational memory as a kernel service: six task tools (`melra_capabilities`,
`melra_plan`, `melra_execute`, `melra_task_status`, `melra_task_cancel`,
`melra_receipt`) and five durable-workflow tools (`melra_workflow_plan`,
`melra_workflow_advance`, `melra_workflow_status`, `melra_workflow_cancel`,
`melra_workflow_control`). pnpm workspace of TypeScript packages (Node 22+, ESM,
strict tsc), plus two Python projects managed by `uv` (`sdk-py`,
`benchmarks/browser-agent`).

## Commands

```bash
pnpm install --frozen-lockfile
pnpm build                  # tsc -p per package; required before tests (see below)
pnpm check                  # versions:check + typecheck + test + python:check — the CI gate
pnpm evals                  # 51 deterministic policy/execution scenarios → evals/results/latest.json
pnpm e2e                    # packages/server/test/e2e.test.ts against a live stdio server
pnpm pack:check              # npm pack --dry-run for the published CLI
pnpm readme:check           # typecheck every ```ts block in every package README
pnpm security:audit          # pnpm audit --prod + scripts/python-audit.mjs
pnpm melra <cmd>            # run the CLI from source via tsx (doctor | setup | init | serve | run | inspect | conformance | clients | workflow | policy test)
```

Single test file (vitest args pass through the package script):

```bash
pnpm --filter @melra/memory test src/index.test.ts
pnpm --filter @melra/memory test -t "ranks exact phrases"
```

Python:

```bash
pnpm python:check           # ruff + pytest for sdk-py
pnpm benchmark:browser:check # ruff + pytest for benchmarks/browser-agent
```

Benchmarks (see README "Reproduce the scores" for the full MiniWoB/LoCoMo invocations):

```bash
pnpm benchmark:core         # builds, then scripts/bench-core.mjs
pnpm benchmark:locomo -- --dataset <locomo10.json> --output <artifact.json>
pnpm benchmark:browser:verify-upstream
```

**Tests import workspace siblings through their `exports` → `dist/`, so `pnpm build` must
run before `pnpm test`** (this is why `typecheck` and `benchmark:*` scripts build first). A
test failing with an unresolved `@melra/*` import means a stale or missing `dist`.

`build`, `typecheck`, and `test` go through `scripts/run-recursive.mjs` rather than calling
`pnpm -r <script>` directly, because every recursive fan-out here multiplies. There is no
vitest config in the repo, so every package defaults to a fork pool of `cores−1`; multiplied
by pnpm's default workspace-concurrency of 4 that is roughly `4 × (cores−1)` Node processes,
each with its own V8 heap. `tsc` has one level instead of two but holds a whole program graph
per process, so four builds in flight is several GiB on its own. Either one is enough to
exhaust memory on a 16 GiB machine. The script sizes the fan-out against RAM and core count
(≈0.4 GiB per test fork, ≈1.5 GiB per `tsc`) and passes it down as `--workspace-concurrency`
plus `VITEST_MAX_FORKS`/`VITEST_MAX_THREADS`. Run `pnpm test:peak-rss` to measure peak
resident memory before and after changing anything about how these are spawned. New recursive
scripts should route through the same runner, not add a second cap.

There is no ESLint/Prettier for TypeScript — `tsc --strict` is the only static gate. Python
uses ruff (line-length 100, py311).

## Execution pipeline (the core invariant)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XAGI-Lab/melra](https://github.com/XAGI-Lab/melra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
