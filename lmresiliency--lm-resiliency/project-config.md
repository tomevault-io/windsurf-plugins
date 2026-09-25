---
trigger: always_on
description: LM Resiliency protects long-running distributed LLM training with two complementary mechanisms:
---

# LM Resiliency Codex Guidance

## Repository purpose

LM Resiliency protects long-running distributed LLM training with two complementary mechanisms:

- **GEMINI** provides frequent asynchronous in-memory checkpoints, peer replication, and fast recovery.
- **SCOUT** detects and localizes silent data corruption (SDC), stragglers, collective desynchronization, process stalls, and selected hardware failures, and participates in checkpoint certification.

Correctness under partial failure is more important than convenience. Prefer conservative behavior when evidence is incomplete or contradictory.

## Working guidance

- Read `CONTRIBUTING.md` before changing code.
- For checkpoint or recovery changes, read `docs/gemini.md` and the relevant tests.
- For detection, replay, consensus, hang, telemetry, or checkpoint-certification changes, read `docs/scout.md` and the relevant tests.
- For public APIs, framework adapters, package imports, or supported versions, read `docs/compatibility.md` and `docs/api.md`.
- Treat this file as review guidance, not as a replacement for tests or the documented runtime contracts.
- Do not duplicate deterministic CI feedback. Ruff, formatting, pre-commit, packaging, and the CPU unit suite are handled by CI.
- Do not weaken a documented safety property merely to make a test pass. Update implementation, tests, and documentation together when the contract intentionally changes.
- Keep optional framework dependencies lazy. Importing `lm_resiliency` must not require DeepSpeed, Megatron Core, TorchTitan, Triton, or CUDA-only packages.

## Pull request review workflow

Use Codex review in batches rather than after every revision commit.

Do not begin editing when the first review comment arrives.

For the current PR head SHA:

1. Wait until the Codex GitHub review has been submitted.
2. Read every unresolved review thread.
3. Run a complete independent review of the branch diff against the base branch.
4. Combine and deduplicate all findings.
5. Fix every accepted finding in one revision, run the relevant tests, and push once.
6. Do not request another broad review until this revision is complete.

- The native Codex GitHub integration handles the initial review when a pull request is opened for review or moved from draft to ready.
- When addressing review feedback, first inspect **all unresolved Codex review threads** and treat the complete set of actionable findings as one fix batch.
- Implement the whole batch, add or update focused regression tests, run the relevant deterministic checks, and inspect the resulting diff before asking for another review.
- Do **not** request `@codex review` after each commit, formatting fix, test-only adjustment, or other intermediate revision.
- Resolve a review thread only after its underlying finding is actually addressed or intentionally rejected with a documented rationale.
- After the current batch is addressed and the relevant CI checks pass, request **one** fresh `@codex review` to look for newly introduced or previously missed issues.
- If that re-review produces new actionable findings, repeat the same batch process and request one additional review only after the next batch is complete.
- A clean later review does not automatically resolve earlier GitHub review threads; verify the old findings are addressed and explicitly resolve those conversations before merge.

## Code Review Rules

Review for concrete correctness, safety, compatibility, and performance regressions. Prefer a small number of high-confidence findings over speculative comments. Do not report style-only issues that deterministic tooling can catch.

Use these severities:

- **P0**: can corrupt training state, select unsafe recovery state, cause widespread data loss, or create a severe security issue.
- **P1**: can deadlock/hang workers, misattribute a failure, break recovery, violate a documented compatibility contract, or introduce a major regression in a supported path.
- **P2**: real but narrower correctness, robustness, test-coverage, or performance issue that should be fixed before relying on the affected path.

For every finding, explain the concrete failure mode and point to the smallest relevant file/line range. Do not raise a finding when the concern is only hypothetical and cannot be tied to reachable behavior.

### Distributed correctness and liveness

- Verify all participating ranks execute compatible collectives in the same order and with compatible tensor metadata.
- Flag one-sided waits, mismatched barriers, lock ordering hazards, background-thread races, unsafe process-group reuse/destruction, and shutdown paths that can strand peers.
- Treat timeout, cancellation, signal, exception, worker-loss, and partial-initialization paths as first-class behavior.
- Check that rank-local decisions do not accidentally become job-wide decisions without consensus, and that job-wide decisions are actually agreed across the required ranks.
- Preserve topology semantics across DP, DDP, FSDP/HSDP, TP, SP, CP, PP, EP, expert TP, ZeRO, and framework-specific process groups when the changed code applies to them.

### GEMINI checkpoint and recovery invariants


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LMResiliency/lm-resiliency](https://github.com/LMResiliency/lm-resiliency) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
