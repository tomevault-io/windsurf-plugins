---
trigger: always_on
description: Telegraph style. Root rules apply throughout the repository. Read the nearest scoped `AGENTS.md` before subtree work.
---

# AGENTS.md

Telegraph style. Root rules apply throughout the repository. Read the nearest scoped `AGENTS.md` before subtree work.

## Start

- Repo: [Crab](https://crab.build) monorepo — serverless Git remote helper. Repositories live in cloud object storage (S3/GCS/Azure), with no Crab data server.
- Replies: repo-root-relative refs only: `crab/src/cmd/push.rs:42`. No absolute paths, no `~/`.
- Fix/triage answers need source, tests, current behavior, and dependency contract proof.
- Reviews/answers: high confidence required. Default to exhaustive relevant codebase search/read, including callers, callees, siblings, tests, docs, and upstream/dependency contracts before verdict. Diff-only review is insufficient.
- Review default: read the whole changed function/module plus callers, callees, sibling implementations, adjacent tests, scoped docs before saying `good`, `bad`, `best fix`, or `proof sufficient`. If challenged, keep reading first; do not defend the earlier verdict until the missing path is checked.
- Before a PR verdict, build a small evidence map: changed surface, entry point, owner boundary, at least one caller and callee, sibling surfaces that share the invariant, existing tests, and current `main` behavior. If any cell is missing, say the gap instead of concluding.
- One-sided fixes need sibling-surface proof, an explanation for why siblings are unaffected, or explicit follow-up work.
- Every PR review must explicitly ask whether the PR is the best fix, not merely a plausible fix.
- Dependency-backed behavior: read upstream docs/source/types first. No API/default/error/timing guesses.
- Missing deps: `make install` (Rust), `npm install` (TS/JS), retry once, then report first actionable error.
- Live-verify when feasible. Never print secrets or AWS creds.
- New `AGENTS.md`: add sibling `CLAUDE.md` symlink; edit `AGENTS.md` only.

## Map

```
CrabBuild/
├── crab/              Rust CLI, remote helper, and product/server composition
├── crates/            Shared Rust contracts, data plane, storage, and orchestration
├── packages/web/      Next.js marketing site and Fumadocs documentation
├── diagram/           Architecture diagrams and rendered assets
├── .github/workflows/ CI, release, service, and evidence workflows
├── .agent/            Repository-local agent workflows
├── .claude/           Repository-local Claude commands and skills
└── .codex/            Repository-local Codex skills
```

Cargo workspace: 20 members — 19 shared crates under `crates/`, plus `crab`.
There is no desktop application, Python package, or SDK package in this workspace; desktop material under `packages/web/` is documentation and marketing content.

## Architecture

### crates (Shared Rust Libraries)

- Scoped guide and subsystem map: `crates/AGENTS.md`.
- Layers: shared contracts → Git/Xet/storage mechanics → metadata/staging/coordination → read/cache/auth/VFS/workflow → server and product composition.
- Low-level crates own reusable contracts and mechanics. Product wiring stays in binaries and product crates; server crates are top-level composition boundaries.
- Public types, feature flags, serialized formats, storage layouts, and errors are cross-crate contracts. Search all consumers before changing them.

### crab (Rust CLI)

- Entry: `crab/src/main.rs` — dual-mode: `git-remote-crab` remote helper + `crab` CLI.
- Subcommands: `crab/src/cmd/` (add, clone, push, hydrate, dehydrate, gc, fsck, mount, status, doctor, and other CLI commands).
- Product modules under `crab/src/` cover command policy, chunking/deduplication, storage, metadata, coordination, Git integration, cache, LFS, auth/read/hydration, tiering/replication, import, and xorb optimization. Reusable contracts and mechanics belong in `crates/`.
- Stack: Rust 2024, tokio, `object_store`, `thiserror`, `tracing`, Blake3.

### packages/web (Next.js)

- Marketing site + Fumadocs documentation. Hosted at crab.build.
- Stack: Next.js 16 (Turbopack), React 19, Tailwind v4, Fumadocs, and shadcn/ui.
- Docs: source content lives under `packages/web/content/docs/`; generated Fumadocs output under `packages/web/.source/` is not hand-edited.
- Follow `AGENTS.md` and adjacent docs/components for web guidance.

### Design Principles

- Fix shape: default to clean bounded refactor, not smallest patch. Move ownership to the right boundary; delete stale abstractions, duplicate policy, dead branches, wrappers, fallback stacks.
- Fix observed local failures with generic product rules; do not hardcode names, ids, log phrases, or user examples in prod code unless they are an explicit contract.
- Compatibility is opt-in. "Shipped" means reachable from a release Git tag; main/GitHub/PR/unreleased code is not shipped.
- Refactor default: one canonical path. Delete the old path unless user explicitly wants compat or the shipped public contract is obvious and cited.
- Config/env surface bar is high. Before adding a config option or env var, first prove existing product behavior, defaults, or doctor migration cannot solve it. Prefer removing or consolidating config/env options when touching these surfaces.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crabbuild/crab-oss](https://github.com/crabbuild/crab-oss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
