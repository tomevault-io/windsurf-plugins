---
trigger: always_on
description: Standalone Vite site and Git-backed incentive protocol for Slop. The public
---

# @elizaos/army

Standalone Vite site and Git-backed incentive protocol for Slop. The public
authorities are `slop.cash` and `slop.tech`; `eliza.army` remains a
compatibility alias.

## Purpose

This package publishes project discovery, installable contributor skills,
separate CI reviewer skills, signed aggregate usage receipts, a live work queue,
project and global leaderboards, permanent contributor profiles, monthly reward
proposals, and verified settlement records. `projects/*/project.json` is the
source of truth for project policy. The generated project and repository
registries must be synchronized from those manifests; never hardcode a project
or target repository elsewhere. This is a private application, not a library.
Cloudflare Pages serves the static build; GitHub Actions refreshes public data
and deploys only after package checks pass.

Each `project.skill.sourcePath` is the one canonical contributor-skill source.
Never maintain a second skill copy. `scripts/prepare-site.mjs` validates every
tracked tree, copies raw Markdown endpoints, builds downloadable `.skill`
archives, and publishes the validated cycle index.

The public checksum is only a corruption check. The generated installer uses
GitHub as an independent trust root: the revision must be current `develop`, a
`develop` ancestor whose complete canonical skill tree is byte-identical to
current `develop`, or an open, non-draft, same-repository PR head into `develop`
with the maintainer-controlled `gitarmy-release-candidate` label. It
recursively
requires the label event to follow the exact current-head commit event, rejects
candidates behind or divergent from current `develop`, and compares the bounded
canonical Contents API file set and immutable raw bytes with the archive.
Working-tree provenance, extra files, and missing files fail closed. Local
versions are immutable sibling directories behind an atomic
relative symlink; a process-bound kernel lock survives interrupted commands
without leaving a stale denial. Updates require an ancestor relationship and
retain the prior verified version. Rollback is explicit: both active and target
trees are byte-verified against GitHub, and the requested target is
reauthorized against current GitHub state immediately before activation. A
canonical per-version authorization receipt preserves the entry-time candidate
PR identity needed to verify a later squash-merge transition; it neither
authorizes rollback nor replaces source-byte verification. Never weaken the
fixed production GitHub origins, the concurrency lock, or the version/symlink
invariants. Tests may inject only deterministic `file://` authorities through
the generator's test option, never environment variables.

## Layout

```text
assets/               repository-owned elizaOS brand assets
projects/             reviewed project manifests and reward policy
skills/               canonical contributor and CI reviewer skills
evaluations/          reviewed awards for useful otherwise-unscored work
cycles/               append-only reward lifecycle records
skill-tests/          bun tests for the bundled skill scripts
src/                  React UI and strict domain/browser contracts
public/               Pages headers/redirects plus generated site assets
scripts/              ingestion, packaging, rewards, settlement, evidence
tests/                unit and real-browser coverage
PRODUCT.md            users, purpose, principles, accessibility
DESIGN.md             visual system and interaction rules
wrangler.toml         Cloudflare Pages Direct Upload contract
```

Generated files under `public/brand/`, `public/downloads/`, `public/projects/`,
and `public/data/cycles/` are produced by `prepare:site`. Do not edit them by
hand.

## Commands

Run from the repository root:

```bash
bun run dev
bun run leaderboard:generate
bun run projects:check
bun run evaluations:check
bun run cycles:check
bun run rewards:close-month -- --cycle YYYY-MM
bun run test
bun run typecheck
bun run lint:check
bun run format:check
bun run build
bun run test:e2e
bun run test:e2e:record
bun run test:e2e:record:production
```

`leaderboard:generate` reads GitHub through the authenticated `gh` CLI or
`GITHUB_TOKEN`; it fails loudly when live data cannot be loaded. The UI keeps
loading, empty, stale, and error states distinct. Never fabricate an empty or
zero leaderboard after an ingestion failure.

The local evidence command builds and records the local preview, but refuses a
missing, empty, malformed, or older-than-eight-hours live ledger. The
production command never rebuilds: it records only the existing `dist`, targets
exactly `https://slop.cash`, byte-compares the deployed skill and ledger
artifacts with that directory, and records DNS, TLS, redirect, and security
header checks. Both modes capture into a fresh sibling staging directory,
validate every artifact and digest, and publish the evidence directory only as
one complete transaction.

## Contribution scoring contract

- Score accepted outcomes, not raw activity.
- Collect and deeply verify the complete rolling 35-day window so a
  first-of-month job can freeze every event in the prior UTC month. Publish the
  exact bounds and record counts; never silently sample.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elizaOS/slopdotcash](https://github.com/elizaOS/slopdotcash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
