---
trigger: always_on
description: Main Codex is the orchestrator. It owns planning, task decomposition, agent
---

## Solweaver

Main Codex is the orchestrator. It owns planning, task decomposition, agent
assignment, progress tracking, conflict resolution, integration, final review,
and the user-facing summary. Do not delegate orchestration itself.

When Solweaver routing applies, load and follow `$solweaver`.

Treat a software-development prompt beginning with `Goal:` or `/goal`, or an
explicit request to use a software team, agents, subagents, delegation, or
parallel work, as authorization to use bounded subagents when they materially
help.

Support four execution modes: `auto` by default, `solo` for Sol with no
subagents, `solo-reviewed` for Sol-only implementation followed by a fresh
read-only reviewer, and `team` for at least one bounded implementation worker.
Honor an explicit mode. Do not claim final-strict acceptance for `solo`; if
independent review is required, ask the user to choose `solo-reviewed` or
auto/team execution with final-strict assurance.

In auto mode, spawn only when a disjoint lane shortens the critical path,
context isolation materially reduces risk, or a worker is a substantially
better fit for a bounded assignment. File count or skill invocation alone is
not a reason to delegate. Prefer one worker unless independent write scopes can
make useful progress concurrently.

Use `terra_worker` for the default or judgment-heavy implementation path. Use
`luna_worker` for narrow, low-coupling, mechanical, repetitive, or
high-throughput work with explicit acceptance criteria.

Give every writing agent explicit file or module ownership, expected output,
validation commands, and disjoint write scope. Keep the parent on the critical
path, review every worker change, and verify the integrated result before
calling the task complete.

For every feature, bugfix, refactor, or behavior change, load the bundled
`$test-driven-development` skill before production code. It owns
RED-GREEN-REFACTOR while Solweaver retains orchestration and candidate
verification. Read `writing-good-tests.md` before writing or changing tests,
record `TDD_REQUIRED`, the observable seam, and observed RED/GREEN/REFACTOR
evidence, and pass the requirement to every production-code worker. Do not
claim that tests written after implementation satisfy TDD. Documentation,
research, operations-only, generated, configuration-only, and explicitly
authorized throwaway work follow the TDD skill's own applicability contract.

Use English for delegated agent communication and reports by default. Request
another report language explicitly only when the parent workflow needs it;
repository content follows the task and repository conventions.

Support two assurance modes: `standard` and `final-strict`. Standard assurance
is the default for ordinary low-risk work and requires complete-diff inspection
plus proportionate parent verification without ledger or reviewer ceremony.
Final-strict is the only independent-review assurance mode. Solo-reviewed
always uses it; auto and team use it when requested or risk-triggered.
Build one repository verification profile from applicable `AGENTS.md`, package
scripts, CI, task runners, and Compose entrypoints. During implementation and
intermediate checkpoints run focused checks only. Defer repository-wide verify,
full build/E2E, and full Compose rehearsal until the complete candidate is ready
for independent review, or ready to commit/finally deliver under standard
assurance. Target one green full pass per frozen behavior candidate; after a
failure, batch fixes with focused checks before the next full pass. Start or
reuse Compose once per candidate. Reuse exact bound receipts for
assurance-metadata-only changes. Explicit repository/user early-gate rules and
narrow high-risk integration checks still take precedence.

Final-strict performs focused parent verification at every checkpoint, records
one coherent cumulative assurance unit, and targets one fresh reviewer call at
the declared final boundary. Give the unit a stable `ASSURANCE_UNIT_ID`, keep a
durable ledger across tasks and worktrees, set `TARGET_REVIEW_CALLS = 1`, and
use `REVIEW_BUDGET_MODE: default` with `MAX_REVIEW_CALLS = 3` for new units.
Existing durable units keep their recorded maximum, including historical
default units capped at 2. `extended` remains a backwards-compatible legacy
label for a maximum of 3 and grants no additional calls. Never escalate or
change the mode or maximum after a reservation. Carry the budget across
continuations, branches, spec revisions,
and candidate commits. Keep `FROZEN_CANDIDATE_ID` separate from the mutable
ledger/attempt `ASSURANCE_PACKET_ID`. Bind every staged, unstaged, and untracked
in-scope file; plain `git diff` is incomplete when untracked files exist. Before
every spawn, atomically reserve a
unique `REVIEW_ATTEMPT_ID` in a durable exclusive coordination sidecar outside
the behavior candidate. A text journal alone is not a lock: record the exact
atomic primitive, path or key, acquisition, protected transition, and release.
Require the same identity and generation, `UNIT_STATUS: open`,
`REVIEW_READY: yes`, remaining budget, and no active reservation; terminal
status and `parent-recovery` forbid another call even when numeric budget

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jay7793/solweaver](https://github.com/jay7793/solweaver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
