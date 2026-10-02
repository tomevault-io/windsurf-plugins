---
trigger: always_on
description: Slop is a GitHub-native incentive network for open-source work. The public
---

# Slop repository instructions

Slop is a GitHub-native incentive network for open-source work. The public
product promise is **make money shipping open source**: contributors use any
agent or model to ship useful work, maintainers accept outcomes in the project’s
own repository, and Slop publishes the score, review state, and verified
settlement record.

This is a private application, not a library. Cloudflare Pages serves the
static site at `slop.cash` and `slop.tech`; `eliza.army` is a compatibility
alias only.

## Product principles

- Score accepted outcomes, not activity.
- GitHub is the work, review, and policy authority.
- Automation proposes; maintainers decide.
- Projected, under-review, approved, scheduled, paid, unclaimed, held, and
  excluded are different states.
- Any provider, model, and agent client may participate when its exact identity
  is disclosed.
- Slop never infers copyright ownership, legal capacity, assignment, wallet
  control, or payment authority.
- Slop never holds keys, signs transactions, broadcasts payments, or claims
  success before public evidence proves it.
- Never publish secrets, prompts, responses, source files, credentials, session
  identifiers, private trajectories, or signing material.

## Source of truth

`projects/*/project.json` is the only project and repository inventory. Never
hardcode a project, repository, reward, steward, status, or funding route
elsewhere. Generated registries and public pages must stay synchronized from
those manifests.

Each `project.skill.sourcePath` is the one canonical contributor-skill source.
Do not maintain a second skill copy. `scripts/prepare-site.mjs` validates the
tree, copies raw Markdown endpoints, builds downloadable `.skill` archives,
and publishes the cycle index.

Generated files under `public/brand/`, `public/downloads/`,
`public/projects/`, `public/protocol/`, and `public/data/cycles/` are build
outputs. Never edit them by hand.

Operational guides under `backend/`, `cycles/`, `disclosures/`, `evaluations/`,
`funding/`, `protocol/`, and `workers/` define subsystem contracts. Keep them focused and
current.

## Repository map

```text
projects/       reviewed manifests and project policy
skills/         canonical contributor and CI reviewer skills
evaluations/    reviewed awards for otherwise-unscored useful work
cycles/         append-only reward lifecycle records
funding/        append-only direct-funding evidence
disclosures/    payouts sent outside the verified settlement flow
protocol/       public attribution and privacy contracts
backend/        private trace storage boundary
workers/        narrowly scoped Cloudflare services
src/            React product and strict browser/domain contracts
scripts/        ingestion, packaging, rewards, settlement, and evidence
skill-tests/    executable tests for bundled skill behavior
tests/          unit, integration, accessibility, and browser coverage
```

## Add a project

The public `/projects/new` route is the preferred starting point. It drafts a
manifest and agent brief, then hands the proposal to GitHub. The website does
not activate a project or create a private admin state.

A complete proposal adds:

```text
projects/<project-id>/project.json
skills/contribute-to-<project-id>/
skills/review-<project-id>-contributions/
```

New projects begin paused. A paused project is registered and listed, but none
of its repositories is collected, so nothing on them reaches the ledger or the
leaderboard. Public contribution access may open independently
when missing authority and terms remain explicit, receipts stay pending, and
payments stay disabled. Verify immutable repository and actor IDs, repository
license facts, GitHub stewardship, integration branch, reward policy, and
failure paths before activating receipts or money states. Stewardship is a
GitHub identity only.

The contributor skill must inspect live GitHub, select bounded unblocked work,
follow the target repository’s rules, test the result, prepare evidence, and
emit the required attribution. It must not claim platform authority over an
issue or create placeholder submissions.

The reviewer skill is separate and advisory. It measures its own run, checks
correctness, tests, security, evidence, duplication, abuse signals, scope, and
usefulness, and places the machine review before the signed attribution footer.

## Installer and attribution

The public checksum detects corruption only. GitHub is the independent trust
root. The generated installer may authorize:

1. current `develop`;
2. a `develop` ancestor whose canonical skill tree is unchanged, or a successful
   approved published revision not listed in `protocol/skill-revocations.json`; or
3. an open, non-draft, same-repository PR head into `develop` with the
   maintainer-controlled `slop-release-candidate` label applied after the exact
   current-head commit event.

Reject candidates behind or divergent from `develop`, missing or extra files,
working-tree provenance, stale label events, mutable redirects, and byte
mismatches. Preserve immutable sibling version directories, the process-bound
kernel lock, atomic relative-symlink activation, prior verified versions, and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SlopDotCash/slopdotcash](https://github.com/SlopDotCash/slopdotcash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
