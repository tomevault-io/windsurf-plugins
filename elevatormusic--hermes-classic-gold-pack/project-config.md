---
trigger: always_on
description: Guidance for AI agents (Codex, Claude Code, etc.) working in this repo. This is
---

# hermes-classic-gold-pack — Agent Guide

Guidance for AI agents (Codex, Claude Code, etc.) working in this repo. This is
an installer/patcher: it applies the Classic Hermes gold theme, Noir Neko pets,
and a custom status bar into a user's real Hermes desktop install. Because it
writes into a live install and into the user's `hermes-agent` checkout, the
overriding value is **do no harm to the user's existing setup**.

## What this pack is

- A Node ESM CLI (`install.mjs`, `update-hermes.mjs`, `scripts/*.mjs`), `type: module`, Node `>=18`.
- It patches/copies files into `HERMES_HOME` and an `apps/desktop` checkout, then records what it did so the change can be updated or undone.
- Three optional tiers live under `advanced/`: `statusbar`, `caduceus`, `watcher`.

## Build, test, lint

- `npm test` → `node --test` (the repo's built-in test runner; test files are `test/*.test.mjs`).
- No build step for the pack itself; `install.mjs` may drive `npm run pack` inside the target `hermes-agent` checkout.
- CI runs `github/super-linter`: **gitleaks** (secrets), **eslint** (JS), YAML/actionlint, bash, and JSON are enforced on changed files. `markdownlint` uses `.github/linters/.markdown-lint.yml` (MD013/MD040/MD022/031/032/033 relaxed). Natural-language and jscpd linters are intentionally off.

## Conventions

- **ESM only** (`.mjs`, `import`/`export`). No CommonJS in pack code (the `advanced/statusbar/files/**` Electron `.cjs` files are payloads copied into Hermes, not pack logic).
- **Dependency injection for testability — reads and time only.** Core functions in `lib/` accept injectable `env`, `platform`, `exists` (reads) and `nowIso` (clock), defaulting to the real ones (see `lib/preflight.mjs`, `lib/hermes-home.mjs`, `lib/pack-stamp.mjs`). New logic should follow this. **The final filesystem *write* is NOT injected:** the record layer calls `writeFileSync`/`cpSync` directly and its tests write to a real temp `HERMES_HOME` (`mkdtempSync`). A direct `writeFileSync` in a write path is the established, correct pattern — do not ask for an injected writer.
- **JSDoc typedefs** on exported functions (params and return shape), matching the existing modules.
- **Corrupt state files are treated as absent**, never fatal — JSON reads are wrapped and fall back (see `readJson` in `lib/pack-stamp.mjs`).

## Invariants — do not break these

1. **Every apply-time write into `HERMES_HOME` is recorded; every delete consumes the manifest.** `lib/pack-stamp.mjs` is the single source of truth for "what's installed" (`hermes-classic-gold-pack.json`) and "how to undo it" (`.manifest.json`). **Concretely: any *apply/staging* write — `writeFileSync`/`cpSync`/`mkdirSync` — whose destination is under `HERMES_HOME` or the `hermes-agent` checkout MUST be paired with `recordApplied()` (stamp) and `appendManifest()` (undo receipt).** A new file or file-type written into `HERMES_HOME` with no matching manifest entry is an **orphan the uninstaller can never remove** — that is data loss, not a nit. **Deletion/uninstall paths (`rmSync`) are the inverse: they reverse a manifest-tracked entry and call `clearApplied()` — they must NOT add a new applied stamp.** Patch, copy, reconcile, and `--no-build` staging all count.
2. **Never silently target an ambiguous install for a write.** An explicit `--home`/`--repo` is authoritative — use it or fail; never fall back to auto-detection (a typo must not hit the user's real install). When auto-resolution is ambiguous (`findHermesHomes` returns more than one), index 0 is a *guess* — confirm before writing.
3. **Uninstall must reverse via the manifest**, not by guessing — the manifest's undo receipts are the contract.
4. **Idempotency.** Re-running an apply must not double-patch or corrupt a partially-applied tier; tier sentinels (`TIER_SENTINELS`) detect an already-applied or reverted state.
5. **Cross-platform paths.** This pack is Windows-first but must not hardcode separators or assume `win32`; use `node:path` and the `platform` param. Compare paths structurally, not as raw strings.

## Review guidelines

When reviewing a PR in this repo, flag (roughly in priority order):

- **P0 — unrecorded write (check this FIRST, before any style/testability comment):** scan the diff for every *apply/staging* write — `writeFileSync`/`cpSync`/`mkdirSync` (and any new file path) — whose destination is under `HERMES_HOME` or the `hermes-agent` checkout. Each MUST be paired with a `recordApplied()` **and** `appendManifest()` call. A write with no manifest entry is an orphan the uninstaller can never remove — flag it as data loss even if the code "works" and the tests pass. Deletes are the inverse: a `rmSync`/uninstall path should consume the manifest and call `clearApplied()` — flag a delete that removes files **not** tracked by the manifest, but do NOT ask a delete to add a new applied stamp.
- **P0 — wrong-target writes:** auto-resolving an ambiguous `HERMES_HOME`/repo and writing to it without confirmation; treating an explicit `--home`/`--repo` as a hint instead of authoritative.
- **P0 — secrets:** any hardcoded token, key, or path containing a real username committed to the tree (gitleaks will also catch this; call it out anyway).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Elevatormusic/hermes-classic-gold-pack](https://github.com/Elevatormusic/hermes-classic-gold-pack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
