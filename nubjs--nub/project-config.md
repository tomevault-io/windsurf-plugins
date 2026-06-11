---
trigger: always_on
description: This file is the entry point for AI coding agents working in this repository. It mirrors the [`AGENTS.md` convention](https://agents.md) used by other AI tools. [`CLAUDE.md`](CLAUDE.md) is a symlink to this file — Claude Code and the `AGENTS.md` convention read identical content.
---

# AGENTS.md — agent orientation for the Nub repo

This file is the entry point for AI coding agents working in this repository. It mirrors the [`AGENTS.md` convention](https://agents.md) used by other AI tools. [`CLAUDE.md`](CLAUDE.md) is a symlink to this file — Claude Code and the `AGENTS.md` convention read identical content.

## Read this first

- This file (orientation, brand-boundary rules, testing philosophy, write-style rules).
- [`wiki/architecture.md`](wiki/architecture.md) — **the single most load-bearing doc.** Nub is a Rust CLI that augments the user's Node via extension surfaces. It is not a fork. Includes the `--node` / `NODE_COMPAT=1` compat-mode contract. Includes the toolchain section. Read this before editing anything under `wiki/`.
- [`wiki/philosophy.md`](wiki/philosophy.md) — the design principles: additivity, brand boundary, reversibility.
- [`wiki/PLAN.md`](wiki/PLAN.md) — canonical design plan; index of all per-feature docs.
- [`wiki/whitepaper.md`](wiki/whitepaper.md) — the framing / pitch / launch post. The current ship-scope intent in user-facing form.
- [`epics/v0.1/todo.md`](epics/v0.1/todo.md) — the v0.1 implementation epic; the to-do list driving the build. Each epic lives in its own subdirectory under `epics/` (the version dimension is in the directory name, not the filename), with a `prompt.md` (framing + how-to-execute) alongside `todo.md` (the phase-by-phase work). Successor epics (`epics/v0.2/`, `epics/v0.1-edits/`, etc.) land as sibling directories.

## Work directly on `main` — no branches, no stashes, no worktrees

**All work happens on `main`.** This is a pre-release product with multiple agents often running in parallel, and the trunk is `main`. Do **not** `git checkout -b`, do **not** `git stash`, do **not** switch branches, and do **not** create git worktrees for isolation. Those operations on a shared working tree cause parallel agents to clobber each other's uncommitted work and orphan commits onto dead branches — this actually happened (2026-05-29: two efforts ended up split across `v0.1-launch-fixes` / `feat/supported-version-expansion-verified` with WIP stranded in a stash, and three stale worktrees locked by a day-old session). The rule that prevents it: **commit small and often, directly to `main`.** Committed work cannot be clobbered; frequent commits shrink the window where loose uncommitted work can be lost to near-zero. If you need to preserve WIP, *commit* it — never stash it. Releases are tag-triggered and deliberate (see [Releasing](#releasing)), so a busy `main` with in-progress work is expected and fine.

## What Nub is, briefly

- **A Rust CLI that orchestrates the user's installed Node.** v0 is a fast script runner plus a TypeScript-just-works runtime. Mechanism: Node's own extension surfaces — `module.registerHooks()`, `--import` preload, `--require`, env vars, N-API addons, V8 flag injection.
- **TypeScript-first developer supertool.** Own transpiler (oxc-based). TS, JSX, non-erasable syntax, `emitDecoratorMetadata` all supported.
- **Compatibility is paramount.** Code targeting Node must run on Nub byte-for-byte. The contract: **default mode = Nub runtime augmentation active; `NODE_COMPAT=1` or `--node` = augmentation disabled, plain Node behavior**.
- **Behavioral additivity** is the rule for augmentation — [`wiki/philosophy.md#additivity`](wiki/philosophy.md#additivity) for the canonical statement. "Additive" means doesn't break code written for vanilla Node; mechanism is restricted to Node's own extension surfaces (not source patches).

## The augmenter / fork distinction is load-bearing

A previous direction (2026-05-17) positioned Nub as a soft fork of Node, with patches to Node source on the table. **That direction was reversed 2026-05-18.** Nub is now a Rust CLI augmenter via Node's extension surfaces only. Source patches are off the table.

If you're editing a plan doc and find "soft fork" or "patched Node" language, it predates 2026-05-18 and should be updated. The authoritative statement of the current direction is [`wiki/architecture.md#augmenter-not-fork`](wiki/architecture.md#augmenter-not-fork).

The reason the distinction is load-bearing for agents specifically: implementation approaches that *sound* clean — "just patch `lib/internal/modules/esm/resolve.js` to do X" — were valid under the old posture and are no longer. The current mechanism rule is **"would a user on plain Node, plus the corresponding `module.register()` / `--import` / npm-addon, get the same result?"** If yes, in scope. If no, find a different mechanism or scope the feature out.

## Design philosophy — read before suggesting features

Before proposing any new feature, API, env var, package, or config surface, read these in order. They are short and load-bearing; do not skip them.

- This file's brand-boundary rules below.
- [`wiki/architecture.md#augmenter-not-fork`](wiki/architecture.md#augmenter-not-fork) — the "would a user on plain Node + the corresponding `module.register()` / `--import` / npm-addon get the same result?" test.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nubjs/nub](https://github.com/nubjs/nub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
