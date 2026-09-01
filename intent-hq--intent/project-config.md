---
trigger: always_on
description: Instructions for AI agents working in this monorepo.
---

# Agent Workflow Guide

Instructions for AI agents working in this monorepo.

## Repository Structure

This monorepo references the Intent component repositories as git
submodules:

- `packages/intentd` → [intent-hq/intentd](https://github.com/intent-hq/intentd) — Rust backend daemon
- `packages/cloudlands-fe` → [intent-hq/cloudlands-fe](https://github.com/intent-hq/cloudlands-fe) — Electron + SvelteKit desktop frontend
- `packages/ios` → [intent-hq/ios](https://github.com/intent-hq/ios) — SwiftUI iOS companion app (private)

Code lives in the submodule repos. The monorepo tracks specific commits of each submodule.
`packages/ios` is private and marked `update = none` in `.gitmodules`: recursive clones and
`git submodule update --init --recursive` skip it by design, and internal devs initialize it
on demand with `make ensure-ios-submodule`.
The durable engineering docs live in `docs/ARCHITECTURE.md` (backend architecture) and
`docs/protocol/` (canonical wire contract); see `docs/README.md` for the docs index.

## Commit & PR Workflow

When changes span a submodule and the monorepo, land the submodule PR (Phase 1); the
monorepo pin advance (Phase 2) then happens automatically.

### Phase 1 — Submodule PRs

1. **Make scoped commits in the submodule** on a feature branch. Group related changes into
   logical commits with conventional commit messages (`feat:`, `fix:`, `chore:`, etc.).
2. **Push the feature branch** in the submodule repo.
3. **File a PR** on the submodule's repo (e.g., `intent-hq/intentd`).
4. **Merge the PR** (squash merge preferred) — **only after explicit permission from a
   human** (see Conventions → Merging). Approved + green checks is not enough.

When the change fixes a monorepo issue, reference it with the full cross-repo form —
`Fixes intent-hq/intent#N` — in the squash-commit message or PR body. GitHub
auto-closes the issue on merge, and the release notifier (see Release Process) comments
on it once a release actually contains the complete fix.

### Phase 2 — Monorepo pin advance (automated)

Submodule pins are advanced automatically by the `auto-bump-submodules` workflow
(`.github/workflows/auto-bump-submodules.yml`): it detects submodule tips ahead of the
recorded pins and lands the bump via a single rolling PR on the `auto/submodule-bump`
branch with auto-merge armed; repeat runs update that PR instead of opening new ones.
The workflow is triggered three ways: each submodule repo notifies the monorepo on push
to `main` via a `repository_dispatch` event (`submodule-update` type), so bumps normally
land within about a minute of a submodule merge; a cron run every 30 minutes acts as a
backstop; and manual `workflow_dispatch` is available for urgent bumps. The
`repository_dispatch` notifications are sent by the submodule repos using the
`MONOREPO_DISPATCH_TOKEN` secret (stored in each submodule repo; a fine-grained PAT with
contents:write on `intent-hq/intent`), and are fail-soft: when the secret is absent
the notify step logs a warning and skips, and the cron backstop still advances the pins.

**Agents (and humans) must NOT file manual submodule bump PRs on the monorepo.** The
workflow owns pin advancement. If an urgent bump is needed, dispatch the workflow
manually instead of filing a PR:

```bash
gh workflow run auto-bump-submodules.yml
```

Regular monorepo PRs for actual content changes (docs, Makefile, CI, scripts) are
unaffected and still follow the normal PR flow.

The workflow authenticates with the `SUBMODULE_BUMP_TOKEN` secret — a fine-grained PAT
with contents:read on `intent-hq/intentd`, `intent-hq/cloudlands-fe`, and
`intent-hq/ios`, plus contents:write and pull-requests:write on `intent-hq/intent`.
Like `INTENTD_RELEASES_TOKEN` / `MONOREPO_ISSUES_TOKEN`, it is fail-soft: when the
secret is absent the workflow logs a warning and exits successfully. The private
`packages/ios` submodule is best-effort — if its tip cannot be read, it is skipped
with a warning and never fails the run.

### Cross-component features (intentd + cloudlands-fe)

For features that need changes in both intentd and cloudlands-fe, development and PR
filing on both repos proceed fully in parallel — nothing serializes until merge time.
Both merges require explicit human permission (see Conventions → Merging), and the
only ordering constraint is the final merge: **do not merge the cloudlands-fe PR
(or arm auto-merge on it, or add it to the merge queue) until the intentd PR is
confirmed merged** — approved/green is not enough. This intentd-first rule applies
specifically to protocol changes (the daemon↔fe wire contract, `docs/protocol/`):
whenever a feature touches the protocol, the daemon side must land first.
Rationale: cloudlands-fe may depend on daemon
behavior/protocol that only exists once the intentd change has landed, so an fe-first
merge can break main or ship against a contract that doesn't exist yet. This rule is
about submodule PR merges, not monorepo bumps — after both are merged, the
auto-bump-submodules workflow advances both monorepo pins automatically (a single
rolling bump PR may cover both submodule refs); do not file a manual bump PR.

## Conventions

- **Conventional commits** are required. PR titles are validated by CI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [intent-hq/intent](https://github.com/intent-hq/intent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
