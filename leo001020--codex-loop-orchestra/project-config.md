---
trigger: always_on
description: <!-- AGENTS.md — Codex LOOP Orchestra discipline section (~100 lines).
---

<!-- AGENTS.md — Codex LOOP Orchestra discipline section (~100 lines).
     This file is a BYTE-STABLE PREFIX: it is loaded at the start of every
     session and must not change between runs of the same package version,
     so prompt caching stays effective. Do not append run-specific content,
     timestamps, or state here — state lives on disk under data/. -->

# LOOP Discipline

## 1. Decomposition discipline (single-pass Plan-and-Solve)

- Planning is a SINGLE PASS: read the task, then emit the complete decomposition
  in one planning event — `packets/*.json` (exactly 4 fields: `goal`,
  `authorized_paths`, `acceptance`, `constraints`, plus `packet_id`) and `dag.json`.
- No incremental re-planning mid-wave. Plans change only at an adjudication
  event (dead-letter, merge conflict, L3 escalation, wave finale).
- Each packet must be self-contained: an Executor with only the 4 fields and its
  worktree can finish it. If a packet needs "context from the main thread", the
  decomposition is wrong — split or rewrite it.
- Intra-wave packets must have non-intersecting `authorized_paths`; the DAG
  assertion enforces this before any dispatch.
- No 5th required packet field. No idempotency_key, no embedded runtime_resources.

## 2. Anti-polling negative discipline (what Sol rounds are NOT for)

Sol is invoked on exactly two event types: **planning** and **adjudication**.
Sol rounds are NEVER used for:

- **Waiting** — native wait-all blocks without producing rounds.
- **Polling** — status lives in `data/events.ndjson`; scripts consume it.
- **Tallying** — report counting is the missing-item check script's job.
- **Retry decisions** — `config/retry_classes.yaml` + retry script decide.
- **State recap** — the state machine holds state; never ask Sol "where were we".

Any transition decidable by if/else, exit code, or a state machine does not go
through an LLM. Sol's ideal round count per task approaches **2 + anomaly count**
(one planning round, one finale round, plus one per genuine anomaly). If a run
uses more, find which of the five forbidden uses leaked back in.

## 3. Return convention (all subagents, mandatory)

- **Success:** exactly 1 line of conclusion + the artifact path(s).
- **Failure:** 1-line conclusion + the last 50 lines of the failing log + the
  report file path.
- Full logs, test output, and exploration notes go to `reports/<packet_id>/`
  on disk — never into the reply body. Self-reported PASS carries zero weight;
  mechanical acceptance replays the commands independently.

## 4. Recoverable compression directive

- Files are the single source of truth; bytes do not reside in Sol context.
- When compressing or compacting: **delete content, keep paths.** A path plus a
  ≤500-token structured summary is always recoverable; inlined content is not.
- Every compaction summary MUST carry: (a) the report index (every report path
  produced so far) and (b) a read/unread checklist so nothing is silently
  dropped. A compaction that loses a path is a defect.
- To re-read detail, use path handles: `head`/`sed` to locate the exact lines,
  never re-ingest whole files.

## 5. Kernel trigger rules (ipybox, when enabled)

Route work to the ipybox persistent kernel when either holds:

- A step will produce **>5,000 tokens of output** — digest it in-process and
  return a variable-name handle plus truncated stdout (≤50 lines printed).
- State must **survive across calls** (dataframes, parsed indexes, counters) —
  kernel state is compression-immune and lives off-heap from the context.

Otherwise stay with plain tools. In this deployed LOOP environment every root and
headless worker can see ipybox, but its Jupyter gateway and IPython kernel start
lazily on the first kernel tool call. Workers that never use it pay no Jupyter
process cost. Kernel discipline: print ≤50 lines, return handles.

## 6. Spawn scale heuristics

- **Homogeneous batch** (same instruction over many rows: per-file review,
  migration checklist, N similar packets): use `spawn_agents_on_csv` with an
  `output_schema` and a stable `id_column`; each worker calls
  `report_agent_job_result` exactly once. After the tool writes
  `output_csv_path`, execute the generated call pack's
  `required_postprocess.argv` and then `required_postprocess.then_argv`.
  The first command copies worktree reports back to LOOP root and emits
  generation-aware terminal events; the second advances the state machine.
  A nonzero postprocess result stops the wave and is never optional.
- **Unique task** (one-off packet, distinct instructions): single `spawn_agent`
  with the packet's 4 fields as the prompt.
- Desktop host slots remain capped by
  `agents.max_concurrent_threads_per_session` (50) per root session, while
  LOOP's normal **cross-plane combined effective target is 80**. A parent
  dialogue sustains 20 agents. The preferred mixed load is execution=60 and
  review=20; these are borrowable reservations, not hard partitions. If one
  pool has no ready work, the other may use all 80 slots.
- Wide execution normally uses WSL/headless `codex exec` workers so Desktop is
  a light control and observation plane. Existing Desktop/CSV entry points stay
  available; routing to headless is a low-friction default, not a global ban.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LEO001020/codex-loop-orchestra](https://github.com/LEO001020/codex-loop-orchestra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
