---
trigger: always_on
description: This repository is for SysKnife, the Linux Agent Control Standard.
---

# SysKnife Operating Notes

This repository is for SysKnife, the Linux Agent Control Standard.
Work here must preserve the trust boundary between the planner,
the shell, and the privileged daemon.

## Current Focus — Ubuntu; GUI Paused

**GUI development is temporarily paused.** Do not review, test, refactor, or
extend `apps/sysknife-shell/**` (the Tauri desktop app). It stays in the tree
and must keep building, so workspace-wide gates still cover it, but it is not
where effort goes. Merging a dependency bump that only touches its lockfile is
fine — that is maintenance, not development.

**Analysis and development focus on Ubuntu**, across its versions (22.04,
24.04, 26.04). Live-VM evidence, action coverage, and security review target
the Debian-family path first.

Fedora is deprioritized, **not** dropped. The rpm-ostree action family, the
atomic story family, and the `FEDORA_ONLY_ACTIONS` safety fence all stay
intact and must keep passing; nothing here licenses deleting them.

## Pre-Commit Gate

**`cargo nextest run --workspace --locked` must pass with zero failures before
any commit.** Fix every failing test — do not commit with known
pre-existing failures. If a test asserts the wrong value, fix the test
assertion to match the code (or fix the code if the test is right); do
not skip or ignore the test.

## Repository Workflow

- Use isolated git worktrees for feature work.
- Prefer one branch per task or tightly related task batch.
- Keep branches small, reviewable, and focused on a single concern.
- Use the PR template for every pull request.
- Open PRs only against `main`.
- Request a code review before merge.
- Apply review fixes, commit them, and push the branch again.
- Merge only after the branch is reviewed and the checks pass.
- **Never leave green PRs hanging.** A PR with all required checks
  passing is ready to merge. Dependabot PRs included — once their CI
  is green, merge them the same day. Open PRs accumulate merge
  conflicts, blow up signal-to-noise, and leave security fixes
  effectively unshipped.
- Delete the remote branch after merge.
- Delete the local branch after merge.
- Delete the worktree directory after merge.

## Worktree Convention

- Keep worktrees outside the repo under
  `~/.config/superpowers/worktrees/sysknife/`.
- Do not leave merged worktrees around.
- Clean up the worktree during branch completion, not later.

## Implementation Standards

- Use test-driven development for behavior changes.
- Write the failing test first.
- Verify the failure before writing implementation code.
- Keep tests close to exhaustive for protocol, daemon, and safety behavior.
- Prefer deterministic unit and integration tests.
- Run the relevant checks frequently instead of waiting until the end.
- Treat regressions as blockers, not polish items.

## CI Expectations

- Keep CI strict enough to catch formatting, lint, and test
  regressions before merge.
- Prefer locked dependency resolution for Rust builds and tests.
- Keep contributor-facing docs lint-clean.
- Keep workflow YAML lint-clean.

## Code Review Expectations

- Review every branch before merge.
- Treat review findings as actionable engineering feedback.
- Fix correctness, safety, and coverage issues before merging.
- Do not merge around an unresolved review finding unless it has
  been proven incorrect.

## Project Boundaries

- `sysknife-brain` plans.
- `sysknife-shell` presents and collects approval.
- `sysknife-daemon` executes privileged actions.
- No component should blur those roles.

## Code Quality

- No dead code. If a workaround is superseded, remove it immediately — do not
  leave it commented out or guarded by a condition that is never true.
- Do not add fallbacks, params, or flags "just in case" — every line of code
  must be reachable and load-bearing.

## Prompt Engineering — System Prompt Rules

The system prompt in `crates/sysknife-brain/src/prompt.rs` is load-bearing.
Changes to it must be validated against the full E2E story suite before merging.

### Per-distro dispatch (PR #203)

`build_system_prompt` dispatches to one of three pure render functions based on
`distro_hint.family`:

- `render_fedora_prompt` — Fedora-family (Silverblue, Kinoite, Fedora Workstation, …)
- `render_debian_prompt` — Debian-family (Ubuntu, Debian)
- `render_generic_prompt` — fallback when no distro hint is available

Each render function concatenates shared `const` blocks with per-distro `const`
blocks. **Fedora prompts never contain Debian action names; Debian prompts never
contain Fedora action names.** This is structural isolation — the model cannot
propose `AptInstall` on a Fedora host or `AddLayeredPackage` on Ubuntu, even if
it hallucinates.

When adding or renaming an action, update the `FEDORA_ONLY_ACTIONS` and
`DEBIAN_ONLY_ACTIONS` string-slice constants that back the safety-fence unit tests.

### The six worked examples are not optional

`prompt.rs` contains Examples A through F. **Do not remove any of them.**

The original Example A — "check disk usage" — was removed; it was a strict
subset of the prose rule and added no measurable coverage. The current examples
are:

- **A** — direct and compound read-only requests
- **B** — installing a package that might already be present (Fedora-path)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lacs-project/sysknife](https://github.com/lacs-project/sysknife) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
