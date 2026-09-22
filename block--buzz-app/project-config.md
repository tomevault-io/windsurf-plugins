---
trigger: always_on
description: Read [the contribution workflow](docs/contributing.md) for commands and validation.
---

# Contributor instructions for AI agents

Read [the contribution workflow](docs/contributing.md) for commands and validation.
For interactive product work, default to edit → human tries the running app →
adjust in the agreed worktree. Do not gate each feedback round on E2E, native
builds, or full validation. Use mandatory hooks and focused behavior checks not
covered by them; use existing CI for broad validation. `just iterate` is optional.
Run `just scan` only when explicitly requested or needed to reproduce a broad
integration failure, not as a routine pre-push or handoff gate.
Track deferred checks: **ready to try** is not **validated**. Check
auth/signing, persistence/migrations, protocol semantics, and destructive writes
before live use.

Files marked `FOUNDATION` require explicit human guidance before editing and
stricter review. Escalate needed changes rather than editing without authorization.

When reviewing CI or test-cost changes, inspect job-summary counts, elapsed wall
time, summed test execution time, and slowest-test/file evidence for regressions.

## Worktree creation

Before creating a worktree, run `git worktree list` and choose the existing
checkout whose local development configuration should be inherited. Immediately
after `git worktree add`, run this from the new worktree:

```sh
scripts/bootstrap-worktree.sh /absolute/path/to/source/checkout
```

Do not start development before bootstrap completes. The script copies the
git-ignored `.env.local` without overwriting an existing target, then uses that
worktree's Hermit proxy to run `bin/pnpm install --frozen-lockfile`. Do not copy
other ignored paths: Keychain credentials and pnpm's package cache are
machine-shared, while dependencies and build output are regenerated. Follow the
per-worktree hook setup in `docs/contributing.md` before committing or pushing.

## Engineering standard

Before editing, state the intended outcome and non-goals. Read the owning code,
callers, and relevant design docs; preserve documented product decisions and
ownership boundaries. Resolve answerable questions from evidence; ask before
deviating from agreed scope or product behavior.

Target **9/10+ for minimalness, elegance, and correctness**: the smallest complete
solution, clear ownership, and no known material defects. Prefer existing patterns
and subtraction. No opportunistic refactors, speculative abstractions, or new
features disguised as fixes. Before expanding into another shared subsystem or
adding alternate-adapter support, show the human the scope change and smallest
complete alternative. Require a current caller or explicit approval for adapter
parity. Review necessity separately from correctness; passing tests do not justify
scope growth. Split at real ownership boundaries, not by deleting safety coverage.

For non-trivial work, make that standard operational:

- Before coding, publish a short scope checkpoint: required behavior, non-goals,
  existing owners/platform support to reuse, expected files, and a rough production
  diff budget (separate from tests/docs). A small fix needs only a sentence, not a
  design ceremony.
- Use one implementation owner per end-to-end change. Reviewers challenge necessity
  as well as correctness; delegate bounded evidence/review, not competing rewrites.
  Review the first working slice before expanding the design, without blocking
  ordinary human UI feedback on a full validation cycle.
- Justify each new abstraction, lifecycle owner, timer, retry policy, or shared
  contract expansion against a current requirement. If the implementation materially
  exceeds the checkpoint, stop adding machinery and show the smallest alternative
  and any behavior tradeoff before continuing. Do not silently weaken agreed behavior.
- Assess the combined feature diff, including stacked PRs. Passing tests, splitting
  PRs, or already-invested work do not establish proportionality. Preserve required
  regression coverage; do not game the budget by deleting tests or compressing code.
  Keep speculative hardening and unrelated failures outside the task.
- Close with one verified end-to-end result and explicit remaining gaps, not a chain
  of green intermediate repairs presented as completion.

Keep files cohesive and group modules and tests by owner. Treat size as a review
signal, not a quota. Extract stable boundaries only when they simplify the
requested change.

Minimal does not mean happy-path-only. Handle relevant boundary inputs, failures,
recovery, and lifecycle transitions; consider concurrency, persistence, security,
and performance where the change affects them. Do not add machinery for
hypothetical requirements.

Validate the affected user contract, not just isolated helpers. Add regression
coverage for changed behavior and relevant failure paths; exercise real integration
boundaries where practical. Follow the contribution workflow's iteration and batch
gates, rather than adding full validation to every edit.

Self-review before handoff; seek independent review for risky changes before
integration. Report what changed, evidence tied to the checked snapshot, and
remaining risks or deferred checks. Green CI is evidence, not proof of user behavior.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [block/buzz-app](https://github.com/block/buzz-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
