---
trigger: always_on
description: This file is the project-level AGENTS.md. Anything stated here OVERRIDES
---

# gwm-cli — house rules for AI assistants

This file is the project-level AGENTS.md. Anything stated here OVERRIDES
defaults and applies to every contribution made via an AI assistant in
this repository.

## 🔴 Primordial rule — Test-Driven Development is mandatory

**No production code lands without a failing test that pinned the
behaviour down first.** This is not a guideline, it is a hard merge
requirement. PRs that add or change behaviour without tests are sent
back, full stop.

### The TDD loop (red → green → refactor)

1. **Red** — write a failing test that captures the new behaviour or
   the bug you are fixing. Run it. It MUST fail for the right reason
   (assertion mismatch, not a compile error in unrelated code). Commit
   the test alone if it helps reviewers see the contract.
2. **Green** — write the minimum production code to make the test pass.
   No extra branches, no speculative abstractions.
3. **Refactor** — clean up while the tests are green. Re-run the full
   suite after every refactor step.

### What counts as "behaviour"

Anything observable from outside the function under test:

- A new CLI subcommand, flag, or output format → end-to-end test in
  `tests/cli_binary.rs` via `assert_cmd`.
- A new public function in `src/<module>.rs` → unit test in
  `tests/<module>_tests.rs`.
- A new bootstrap step (file copy, guard, no-symlink, command hook) →
  integration test in `tests/bootstrap_tests.rs` exercising it against
  a `tempfile::TempDir`.
- A libgit2 worktree operation → integration test in
  `tests/worktree_integration.rs` using `tests/common::init_repo()`.
- A TUI state transition → state-machine test in
  `tests/tui_app_tests.rs` (ratatui-free).

### Exceptions (narrow, must be argued in the PR description)

The bar to skip a test is "the change is observably untestable from
the public surface". Concretely:

- **Pure formatting / typo fixes** in user-facing strings → no test
  required if the string is incidental (a log line, a help blurb). If
  the string is asserted somewhere, update the assertion.
- **Dependency bumps** without behaviour change → CI green is the test.
- **Comments-only changes** → no test required.

Everything else needs a test. "I tested it manually" is not an
exception; codify the manual test as an integration test.

### Enforcement

- PR template ships with a `cargo test` checkbox under **Tests**. Do
  not tick it unless the suite actually ran green locally.
- Reviewers will run `git log --stat <branch>..HEAD -- tests/` and
  block the PR if the touched module has no companion test diff.
- `tests/cli_binary.rs::help_prints_subcommands` should be updated
  every time a new subcommand is added — treat this as the canary.

## Other house rules

- **Reconcile open PRs before any tag.** Before cutting an RC or a
  stable, run `gh pr list --state open` and account for every open
  PR: either it's in the changeset, intentionally deferred, or
  closed as stale. The v0.3.0 stable shipped without three queued
  feature PRs (#51, #52, #53) because this check was skipped —
  forced an immediate v0.4.0 promotion 38 minutes later. Two
  minutes upfront beats a rushed follow-up release.
- **Release notes are per-version, never the index.** The release
  workflows (`release.yml` / `pre-release.yml`) source their
  `body_path` from `changelogs/<version>.md` (stable) or
  `changelogs/pre-releases/<version>.md` (rc/alpha/beta), NOT from
  the top-level `CHANGELOG.md` (which is the in-progress index —
  entries get moved into the per-version file when the release is
  cut, so the index is empty at tag time). Before tagging, verify
  the per-version file exists and contains the release contents;
  the workflow now hard-fails if the file is missing rather than
  silently publishing the empty index (witnessed on v0.6.0 /
  v0.6.0-rc.1 — both releases had to be re-edited post-hoc via
  `gh release edit --notes-file`).
- **Do not stack deep PR chains.** After the v0.7.0 hardening run,
  the cost of rebasing stacked PRs was higher than the cost of waiting
  for review. For decompositions touching related surfaces, keep at
  most 2-3 PRs open, merge each PR as soon as Copilot + CI are green,
  wait for `dev` to settle, then branch the next one.
- **Batch low-risk encapsulation nits after a stack merges.** If a
  Copilot review asks for private fields, accessors, or re-export
  cleanup while several dependent PRs are queued, prefer filing or
  applying a small polish PR after the stack lands. Do not force a
  cascade of mechanical rebases for non-behavioural cleanup.
- **Parallel agents only when file ownership is disjoint.** Sub-agents
  work well for independent surfaces. If multiple tasks all touch
  shared files such as `src/tui/app.rs`, `src/tui/state/*`, or config
  plumbing, dispatch them sequentially instead of creating avoidable
  merge conflicts.
- **Follow-up issues beat scope creep.** If review uncovers a design
  bug whose fix changes the shape of the implementation, file a
  focused follow-up issue rather than hiding it in the current PR.
  Keep the original PR atomic unless the bug invalidates its contract.
- **Verify MSRV against the whole codebase, not the feature you just
  added.** Before declaring or changing MSRV, run `cargo clippy

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kbrdn1/gwm-cli](https://github.com/kbrdn1/gwm-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
