---
trigger: always_on
description: This file is the **index** to the project's canonical record. Every session:
---

# AGENTS.md — vllm.cpp canonical index

This file is the **index** to the project's canonical record. Every session:
read this first, follow the links that matter for your task, and keep the
record updated (append to the state log) — commit it with your changes.
Push directly to `main`.

**Keep `README.md` (the user-facing status) CURRENT at EVERY feature/iteration
checkpoint.** In the SAME change that advances a spike, implementation, test,
gate, benchmark attempt, or lifecycle state, update the matching README
section/table row with the exact current stage — including `ACTIVE`/`GATING`,
failed or void runs, and explicit pending hardware work. Do not wait for a
feature to land or a gate to pass. Keep its ⚠️ header, architecture /
acceleration / quantization tables, and "Status & caveats" mutually consistent.
The README must never lag reality and must not turn progress into support.

**Keep [`docs/BENCHMARKS.md`](docs/BENCHMARKS.md) CURRENT at the SAME
checkpoint.** Every feature/iteration records its benchmark disposition there
in the same change: accepted numbers with exact workload/reference/evidence,
or an explicit `PENDING`, `NOT APPLICABLE`, `FAILED`, or `VOID` reason and the
next reproduction command. Never publish a partial/contended/stale-denominator
number as binding. `scripts/check-doc-checkpoint.py` and its CI job enforce that
every code/test/benchmark/spike/lifecycle commit updates both public checkpoint
surfaces; do not weaken the checker to bypass the obligation.

**Keep the ROADMAP (`.agents/roadmap_v1.md`) and its AREA MATRICES CURRENT —
same-change obligation.** The roadmap is the single top-level portfolio table;
the linked engine/model/quantization/kernel/backend matrices are the detailed
execution-status surfaces; `feature-matrix.md` remains the broad parity coverage
view. Any change that shifts a feature's or track's state updates
the owning matrix row (status, spike/spec, implementation + test evidence) and
the roadmap portfolio row **in the SAME change**, exactly like the README rule
above. Neither is ever updated speculatively: `DONE` means merged and gated,
with non-empty code and test anchors. Applies to every sub-agent; reviewers
treat a state-shifting diff without its matrix/roadmap update as incomplete.

**Doc lifecycle — live context vs completed record (user-directed 2026-07-10).**
`.agents/` holds documents that are LIVE context for current work; era-closed
documents move to **`.agents/completed/`** (version/era-stamped name, e.g.
`completed/roadmap_mvp_v0.md`) in the same change that closes their era, with
all repo links fixed. The roadmap is VERSIONED: `roadmap_v1.md` is current;
when superseded, it moves to `completed/` and `roadmap_v2.md` takes its place.
Nothing under `completed/` may be load-bearing for live decisions — if you need
to cite it for current work, the relevant content belongs (summarized) in a
live doc. Rationale: a reader of `.agents/` should see exactly what bears on
what we are doing NOW, nothing stale mixed in.

**Spec/scoping location.** All feature-specific implementation specs, scoping
reports, semantics notes, feasibility studies, and design references live under
`.agents/specs/`, never at the `.agents/` top level. The top level is reserved
for the live project-wide protocol, roadmap, status, environment, inventory,
and ledger. Specs that cease to be live context follow the same lifecycle and
move to `.agents/completed/` with their links repaired.

## STANDING DIRECTIVE — tabular inventory, spike first, then parallel claims

The canonical record is **table-first**. Before implementation begins, enumerate
the complete upstream surface in the owning area matrix. Every row has a stable
ID and these fields: upstream source, our implementation anchor, tests/evidence,
spike/spec, lifecycle state, and owner/claim. The canonical area matrices are:

- `.agents/engine-matrix.md` — stable execution rows for engine, KV, scale-out,
  sampling, serving, and other cross-cutting behavior;
- `.agents/feature-matrix.md` — broad one-by-one parity coverage view; it rolls
  up to the engine matrix and the domain matrices below;
- `.agents/model-matrix.md` — every model architecture/family registered by the
  pinned vLLM;
- `.agents/quantization-matrix.md` — every tracked storage/quantization scheme,
  separated by loader, dequant/compute path, backend, and end-to-end gate;
- `.agents/kernel-matrix.md` — vLLM and dependency kernel families plus our
  dispatch/architecture coverage;
- `.agents/backend-matrix.md` — platform and CUDA-architecture targets plus the
  native competitor/performance gate for each backend.

**Every item is spiked before it is implemented.** Its committed
`.agents/specs/<slug>.md` spike must inventory the whole upstream/dependency
chain, dispatch rules, exact files to port, tests to port, hardware needs,
correctness/performance gates, dependencies, and a row-sized work breakdown.
No row may enter `READY`/`ACTIVE` without that spike. An implementation already
present in the tree is not allowed to claim `DONE` in a matrix until the row
links exact code and test/evidence anchors, its ledger evidence, and the closing
commit; record gaps honestly as `ANCHOR-BACKFILL` or `PARTIAL`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mudler/vllm.cpp](https://github.com/mudler/vllm.cpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
