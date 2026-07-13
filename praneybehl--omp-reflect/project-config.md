---
trigger: always_on
description: `omp-reflect` is a standalone [oh-my-pi](https://github.com/can1357/oh-my-pi) extension. It is developed against the **published** `@oh-my-pi/*` packages pinned in `package.json` — never against a workspace catalog or `bun link`. A fresh clone must build with nothing but `bun install`.
---

# Development Rules

`omp-reflect` is a standalone [oh-my-pi](https://github.com/can1357/oh-my-pi) extension. It is developed against the **published** `@oh-my-pi/*` packages pinned in `package.json` — never against a workspace catalog or `bun link`. A fresh clone must build with nothing but `bun install`.

## Commands

```bash
bun install --frozen-lockfile   # install (lockfile is committed)
bun run check                   # biome check . && tsgo --noEmit
bun test                        # full suite; must stay green
```

Never use `tsc`/`npx`; this is a Bun-only project (Bun APIs over `node:*` equivalents, `bun:sqlite`, `Bun.file`/`Bun.write`).

## Hard constraints

These are load-bearing invariants. Breaking any of them is a bug even if every test passes.

### Wire contract

- `src/wire.ts` is a **verbatim mirror** of the host's `packages/stats/src/reflection-wire.ts` (constants, literal types, payload shapes, `ACTIVITY_REFLECTION_SCHEMA_VERSION = 1`). Any change must land in the host first; `test/wire.test.ts` imports the host file by relative sibling path (`../../oh-my-pi/...`) and fails on drift.
- The sidecar filename is exactly `__omp-reflect.jsonl`, written beside the audited session's artifacts. One `start` entry per attempt before dispatch, exactly one `finish` after — statuses `success | invalid | provider_error | aborted` only.
- Persist reported provider usage on every finish that has it (failed responses are billed activity). Never persist raw task excerpts or raw provider error text.

### Host facade boundary

- The host's stats database is optional and is accessed **only** through a present `ctx.stats` five-method facade mirrored in `src/host-stats.ts`. No file in this repo may value-import `@oh-my-pi/omp-stats` aggregator, db, or gain modules — type-only imports from `@oh-my-pi/omp-stats/types` are the sole allowed dependency on that package.
- `requireHostStats()` must keep throwing the exact string `Activity Reflections requires an oh-my-pi build with ctx.stats.` for callers that explicitly require that facade. Reflection observability itself must instead prefer a valid host facade and otherwise use the injected standalone source.
- Host-only behavior matrices and Snapcompact gain are never inferred from transcript text. Standalone mode may provide only its own model/tool aggregates; its behavior and gain payload sections stay empty.

### Extension-owned activity analytics

- The only activity store is `${getAgentDir()}/omp-reflect-activity.sqlite`. Never open, attach, read, write, migrate, copy, or inspect the host `~/.omp/stats.db` (or any host `stats.db`) directly. The extension-owned database is intentionally independent even when `ctx.stats` exists.
- `ActivityDb` lease invariants are transactional: claim, renew, and release use `IMMEDIATE` transactions; every mutation that applies parsed facts or reconciles missing owners verifies the live owner lease before its first write; a stale owner throws before changing offsets or facts. Sync keeps the lease alive with its heartbeat and releases only its own lease.
- Port parser and aggregate behavior from oh-my-pi branch `feat/activity-insights`, not a new transcript interpretation. Preserve the post-review handling for custom-message skill prompts, rejected `?#` suffixes, monotonic read confirmation, image-only tasks, nested tool-result timestamps, reflection sidecars, and agent-kind path classification. Record any required published-16.3.15 API adaptation in the implementation report.
- The local `/activity` dashboard reads only `ActivityDb`; its server and tests use injected DB/runtime seams and never depend on a real agent directory.

### Payload bounds & sanitization

- Bounds are contractual: user prompt ≤ 2,000 chars, final assistant answer ≤ 3,000, complete payload including observability JSON ≤ 24,000; ≤ 6 task windows per attempt; observability matrices bounded to top-8/8/12/12 with one slot reserved for the active model.
- Reflection is an **ongoing, watermarked process**: BOTH manual and scheduled runs select only windows whose `sourceEntryIds` are not yet covered by a successful attempt in the session's sidecar — the sidecar IS the durable watermark, and six is the per-attempt batch bound, not the coverage horizon. A fully covered session is the caught-up steady state: notify, dispatch nothing, record no attempt (never burn the retry floor on it). `/reflect status` must keep reporting the watermark (`covered/total` + insights-through timestamp).
- Excluded from the payload, always: tool arguments/results, images, system prompts, hidden custom message bodies, subagent text.
- Every excerpt passes `createReflectionSanitizer()` (fresh secret reload per dispatch, control-delimiter neutralization, secret obfuscation) **before provider transmission and before persistence**. Reflection output is never deobfuscated.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [praneybehl/omp-reflect](https://github.com/praneybehl/omp-reflect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->
