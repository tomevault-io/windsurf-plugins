---
trigger: always_on
description: Atlas's entry point is a frontier model (Fable, or whatever the best frontier model is)
---

# Claude Code Instructions — Atlas

## The end goal (north star — test every change against this)

Atlas's entry point is a frontier model (Fable, or whatever the best frontier model is)
acting as the **driver**. The driver understands the user's intent using skills built for
it, then — inside a loop — decomposes the goal into optimal components and routes each to
the **cheapest capable executor**, to save as many tokens as possible while delivering an
outcome as good as if the frontier model had done all the work itself, at a fraction of
the cost.

The executor pool is **any CLI agent harness** (Claude Code, Codex, Gemini, whatever
ships next) **plus direct API paths** (the native backend's parse/expand/rate), all in
parallel. Not a fixed provider list. Every feature decision is tested against: *does this
make the frontier-driver-decomposes-to-cheap-executors loop better, cheaper, or more
provable?*

What makes Atlas a product, not a prompt pack (the four moats):
1. **Cross-vendor arbitrage** — route work across any harness/API by cost and capability.
2. **Engine-enforced unfakable gates** — validate-tasks, ship-check (`SHIP_CHECK_OK`),
   the two-axis "done" (oracle + reachability, below), structural tests. If a feature could
   be a skill, it's not a product feature until the engine enforces it.
3. **Persistent vendor-neutral task graph** — tasks.json survives across vendors/sessions.
4. **Cross-vendor cost ledger** — telemetry + economy-report prove the savings.

## Unfakable "done" — the two gates (5.3.0+, the trust backbone)

A task ships only when **both** axes pass — `done = oracle-PASS AND code WIRED/EXEMPT`.
This is the property the whole product (and the marketplace below) rests on; do not weaken it.

- **Gate 5 — re-execution oracle.** The binding ship gate is `skel/ship-check.py`
  (oracle-backed). It re-runs the operator-held tests at the *claimed commit* in a
  network-isolated, digest-pinned **podman** sandbox, so a submitter cannot pass by editing
  their own logs. `prd_taskmaster/oracle_bridge.py` maps a CDD card → graded verdict via the
  spine's `atlas oracle` CLI; it is **fail-closed** (any ambiguity → FAIL, never PASS). The
  self-grantable `SHIP_CHECK_OVERRIDE_ADMIN` bypass was removed.
  (`prd_taskmaster/shipcheck.py` is a NON-binding display heuristic only — never add an oracle
  call there.)
- **Gate 6 — reachability** (`prd_taskmaster/reachability.py`, `reachability-sweep` CLI). A
  green test on an orphan module (imported by nothing) is **blocked** and surfaced as
  `⚠ scaffolded`. Verdicts: WIRED (a non-test file imports it) / EXEMPT (declared
  `cli:|route:|tool:|hook:|plugin:|dynamic:` scheme, accepted on trust in v1) / ORPHAN (fail).
  Read-only at runtime.

## Tournament marketplace (`prd_taskmaster/tournament/`)

The trustworthy two-gate signal feeds a **settled tournament**: N executors race one job →
every submission is adjudicated through both gates → the winner is paid in **AtlasCoin** →
a UCB reputation store routes the next job to the cheapest proven-capable executor.

- `tournament-run` / `tournament-status` CLI; flow is `spawn → collect → adjudicate → settle
  → reputation` (`cmd.py`), with anti-sybil slots released in a crash-safe `finally`.
- `collect.py` is the security core — **commit-reveal** (`commit_hash = sha256(diff base..HEAD)`)
  defeats the diff-copy attack. `antisybil.py` enforces per-job / per-operator economic caps.
- `goose_backend.py` is the cheap-API racer (one OpenRouter model via `goose run`).
- `watcher.py` (`watcher-run` / `watcher-status`) is the **independent out-of-band re-execution
  watcher** — the precondition for real slashing. It re-adjudicates settled submissions from
  primary evidence (re-runs the oracle, independently re-derives `sha256(diff base..HEAD)` to
  catch diff-copy), abstains when it cannot verify (never a false confirm), and **engine-gates**
  real forfeiture: `run_tournament` downgrades `--enforce-slash` to shadow unless the fail-closed
  `permit_enforce_slash` confirms the whole job behind a concordance track record.
- **Invariants (enforced + tested — keep them):** **shadow-slash by default**; real slashing only
  behind the watcher's fail-closed permit (no AtlasCoin burned without an independent positive
  confirmation); honest losers are **always refunded**; **AtlasCoin is conserved** (no mint/burn,
  even under account aliasing).

## The spine (`atlas-protocol`, separate repo)

The gates shell out to the **spine CLI** (`atlas oracle` / `atlas tournament settle` / ledger),
which lives in the `atlas-protocol` repo (branch `dev`) and runs via `tsx` today. Point
`ATLAS_ORACLE_CMD` at it (e.g. `tsx apps/cli/src/index.ts`) for the real-podman e2e gates.
Packaging the spine for production is on the roadmap — see CHANGELOG `## [5.3.0]`.

## Backend model (v4.1+)

TaskMaster is one pluggable backend, not a prerequisite. *"Atlas speaks TaskMaster
natively — but doesn't need it."*

- Backend resolution: `backend: auto|taskmaster|native` in `.atlas-ai/fleet.json`
  (auto = taskmaster iff binary ≥0.43 present, else native). The 5-op protocol
  (detect / init_project / parse_prd / expand / rate) lives in `prd_taskmaster/backend.py`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anombyte93/prd-taskmaster](https://github.com/anombyte93/prd-taskmaster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
