---
trigger: always_on
description: Starcite is a clustered Phoenix application that provides durable, low-latency session event storage for LLM applications. It maintains ordered session histories with sub-150ms p99 appends via Raft consensus, leaving prompt construction and token management to the client.
---

# Starcite - Agent Guide

Starcite is a clustered Phoenix application that provides durable, low-latency session event storage for LLM applications. It maintains ordered session histories with sub-150ms p99 appends via Raft consensus, leaving prompt construction and token management to the client.

## Ground Rules

- Use the pinned local toolchain from `.mise.toml`. Run `mise install` after
  cloning, then `mise trust .mise.toml`, then either activate `mise` in your
  shell or prefix local project commands with `mise exec --`.
- Run `mise exec -- mix deps.get` in fresh worktrees before compile, test, or
  precommit commands.
- Run `mise exec -- mix precommit` before you hand work back unless your shell
  already has `mise` activated. It compiles with warnings-as-errors, formats,
  and runs tests.
- Use the built-in [`Req`](https://hexdocs.pm/req/Req.html) client for HTTP. Do not add `:httpoison`, `:tesla`, or `:httpc`.
- Never introduce new dependencies or services without explicit approval.
- Keep the default Tailwind v4 imports in `assets/css/app.css`; extend styling with Tailwind utility classes, not `@apply`.
- Do not add decorative separator/banner comments (for example `# -----`); keep comments meaningful and specific.

## Golden Patterns

- Fail loudly on bad input. Prefer function-head pattern matching or `with` pipelines that enforce required keys. Only provide defaults when the product intentionally supports omissions.
- Validate telemetry metadata where the metric contract is defined (typically the telemetry helper or boundary module). Do not duplicate the same validation in callers.
- Destructure known structs/maps and coerce once. Avoid chaining `Map.get/3` with defaults-trust shape where it’s guaranteed, and guard upstream.
- Honour immutability: produce new assigns/state rather than mutating in place, and prefer small pure helpers for transformations.
- Design for back-pressure. The runtime assumes at-least-once delivery and sequence numbers; handle drains and retries with clear status signaling.

## Simplify First

- Start with the simplest readable implementation that works, then add complexity only when a measured requirement demands it.
- Do not duplicate parsing/validation across layers. Parse and coerce at system boundaries (for example `runtime.exs`), then trust internal types.
- Prefer direct data flow over defensive transformation pipelines. Avoid re-shaping maps multiple times when one representation already fits.
- For storage formats, prefer off-the-shelf encoding/decoding (`Jason` for JSON/JSONL) over custom codecs unless there is a concrete performance or compatibility need.
- Keep adapters thin: isolate transport/client calls from layout/serialization concerns, but avoid fragmenting logic into many tiny helper modules without clear payoff.
- Remove generic “just in case” guards in internal paths. Crash loudly on impossible states rather than silently normalizing them.
- Never introduce `maybe_*`, `normalize_*` functions, or pass-through helpers that only forward to another function without changing representation or behavior.
- Minimize bespoke parsing (regexes, hand-rolled XML/date parsing, etc.) unless required by an unavoidable external protocol.
- Optimize for fewer lines and clearer control flow. Deleting code is preferred to adding abstraction when behavior stays correct.
- For prototype work, bias toward readability and explicitness over hardening.

## Domain Assumptions

- Messages are append-only and replayable with deterministic sequence numbers.
- Sessions are stored durably in 256 Raft groups (3 replicas each, quorum writes).
- No compaction, token budgets, or prompt-window logic—clients own that responsibility.
- Background flusher streams Raft state to Postgres (non-blocking, idempotent).

## Performance Learnings

### Generic Hot-Path Playbook (Agent Reusable)

- Step 1: define the target before edits (`qps`, `p95`, `p99`, timeout/error rate, CPU, allocations).
- Step 2: run a baseline and record both offered load and effective throughput.
- Step 3: optimize one bottleneck class at a time, then rerun the same workload.
- Step 4: keep changes only when they improve the declared target; revert complexity otherwise.

- Signal: high allocation rate on a dominant single-item path.
- Action: add a dedicated single-item code path that bypasses batch list building, reverse, and map churn.
- Guardrail: preserve ordering and idempotency semantics exactly.
- Exit criterion: lower alloc pressure and higher sustained throughput at equal or better latency.

- Signal: endpoint/controller CPU dominates even before consensus/storage.
- Action: parse and validate once at the boundary using strict fast clauses; keep a separate slow path for optional inputs.
- Guardrail: invalid input still fails loudly with explicit errors.
- Exit criterion: reduced boundary CPU without shape/validation regressions.

- Signal: hot path performs non-essential synchronous reads.
- Action: skip read-path work when authorization or flow type makes the read unnecessary.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fastpaca/starcite](https://github.com/fastpaca/starcite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
